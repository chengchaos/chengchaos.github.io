---
title: Install Fabric and Fabric Samples
key: 2021-12-27
tags: Fabric
---



参考： [https://hyperledger-fabric.readthedocs.io/en/latest/install.html](https://hyperledger-fabric.readthedocs.io/en/latest/install.html)

<!--more-->

我们认为实践出真知，因此为了帮助大家使用 Fabric，我们创使用 docker compose 建了一个简单的 Fabric 测试网络和一个简单应用集合来示范 Fabric 的核心能力。我们也预先编译了 `Fabiric CLI tool binaries` 和 `Fabric Docker Images` 可以下载到您的环境中。



## 下载 Fabric 示例，docker 镜像和二进制文件

下载 `fabric-samples` 到 `$HOME/go/github.com/<your_github_userid>` 目录中。这是 Go 语言社区推荐的 Go 工程的做法。如果您使用不同的 `GOPATH` 目录或者您使用 Windows 操作系统，看这个 [说明](https://hyperledger-fabric.readthedocs.io/en/latest/install.html#notes) 。



```bash
mkdir -p $HOME/go/src/github.com/chengchaos
cd $HOME/go/src/github.com/chengchaos
```

下载最新版本的 Fabirc sample， docker 镜像和二进制文件。

```bash
$ curl -sSL https://bit.ly/2ysbOFE | bash -s
```



bootstrap.sh



```bash
#!/bin/bash
#
# Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#

# if version not passed in, default to latest released version
VERSION=2.4.0
# if ca version not passed in, default to latest released version
CA_VERSION=1.5.2
ARCH=$(echo "$(uname -s|tr '[:upper:]' '[:lower:]'|sed 's/mingw64_nt.*/windows/')-$(uname -m | sed 's/x86_64/amd64/g')")
MARCH=$(uname -m)

printHelp() {
    echo "Usage: bootstrap.sh [version [ca_version]] [options]"
    echo
    echo "options:"
    echo "-h : this help"
    echo "-d : bypass docker image download"
    echo "-s : bypass fabric-samples repo clone"
    echo "-b : bypass download of platform-specific binaries"
    echo
    echo "e.g. bootstrap.sh 2.4.0 1.5.2 -s"
    echo "will download docker images and binaries for Fabric v2.4.0 and Fabric CA v1.5.2"
}

# dockerPull() pulls docker images from fabric and chaincode repositories
# note, if a docker image doesn't exist for a requested release, it will simply
# be skipped, since this script doesn't terminate upon errors.

dockerPull() {
    #three_digit_image_tag is passed in, e.g. "1.4.7"
    three_digit_image_tag=$1
    shift
    #two_digit_image_tag is derived, e.g. "1.4", especially useful as a local tag for two digit references to most recent baseos, ccenv, javaenv, nodeenv patch releases
    two_digit_image_tag=$(echo "$three_digit_image_tag" | cut -d'.' -f1,2)
    while [[ $# -gt 0 ]]
    do
        image_name="$1"
        echo "====> hyperledger/fabric-$image_name:$three_digit_image_tag"
        docker pull "hyperledger/fabric-$image_name:$three_digit_image_tag"
        docker tag "hyperledger/fabric-$image_name:$three_digit_image_tag" "hyperledger/fabric-$image_name"
        docker tag "hyperledger/fabric-$image_name:$three_digit_image_tag" "hyperledger/fabric-$image_name:$two_digit_image_tag"
        shift
    done
}

cloneSamplesRepo() {
    # clone (if needed) hyperledger/fabric-samples and checkout corresponding
    # version to the binaries and docker images to be downloaded
    if [ -d test-network ]; then
        # if we are in the fabric-samples repo, checkout corresponding version
        echo "==> Already in fabric-samples repo"
    elif [ -d fabric-samples ]; then
        # if fabric-samples repo already cloned and in current directory,
        # cd fabric-samples
        echo "===> Changing directory to fabric-samples"
        cd fabric-samples
    else
        echo "===> Cloning hyperledger/fabric-samples repo"
        git clone -b main https://github.com/hyperledger/fabric-samples.git && cd fabric-samples
    fi

    if GIT_DIR=.git git rev-parse v${VERSION} >/dev/null 2>&1; then
        echo "===> Checking out v${VERSION} of hyperledger/fabric-samples"
        git checkout -q v${VERSION}
    else
        echo "fabric-samples v${VERSION} does not exist, defaulting to main. fabric-samples main branch is intended to work with recent versions of fabric."
        git checkout -q main
    fi
}

# This will download the .tar.gz
download() {
    local BINARY_FILE=$1
    local URL=$2
    echo "===> Downloading: " "${URL}"
    curl -L --retry 5 --retry-delay 3 "${URL}" | tar xz || rc=$?
    if [ -n "$rc" ]; then
        echo "==> There was an error downloading the binary file."
        return 22
    else
        echo "==> Done."
    fi
}

pullBinaries() {
    echo "===> Downloading version ${FABRIC_TAG} platform specific fabric binaries"
    download "${BINARY_FILE}" "https://github.com/hyperledger/fabric/releases/download/v${VERSION}/${BINARY_FILE}"
    if [ $? -eq 22 ]; then
        echo
        echo "------> ${FABRIC_TAG} platform specific fabric binary is not available to download <----"
        echo
        exit
    fi

    echo "===> Downloading version ${CA_TAG} platform specific fabric-ca-client binary"
    download "${CA_BINARY_FILE}" "https://github.com/hyperledger/fabric-ca/releases/download/v${CA_VERSION}/${CA_BINARY_FILE}"
    if [ $? -eq 22 ]; then
        echo
        echo "------> ${CA_TAG} fabric-ca-client binary is not available to download  (Available from 1.1.0-rc1) <----"
        echo
        exit
    fi
}

pullDockerImages() {
    command -v docker >& /dev/null
    NODOCKER=$?
    if [ "${NODOCKER}" == 0 ]; then
        FABRIC_IMAGES=(peer orderer ccenv tools)
        case "$VERSION" in
        2.*)
            FABRIC_IMAGES+=(baseos)
            shift
            ;;
        esac
        echo "FABRIC_IMAGES:" "${FABRIC_IMAGES[@]}"
        echo "===> Pulling fabric Images"
        dockerPull "${FABRIC_TAG}" "${FABRIC_IMAGES[@]}"
        echo "===> Pulling fabric ca Image"
        CA_IMAGE=(ca)
        dockerPull "${CA_TAG}" "${CA_IMAGE[@]}"
        echo "===> List out hyperledger docker images"
        docker images | grep hyperledger
    else
        echo "========================================================="
        echo "Docker not installed, bypassing download of Fabric images"
        echo "========================================================="
    fi
}

DOCKER=true
SAMPLES=true
BINARIES=true

# Parse commandline args pull out
# version and/or ca-version strings first
if [ -n "$1" ] && [ "${1:0:1}" != "-" ]; then
    VERSION=$1;shift
    if [ -n "$1" ]  && [ "${1:0:1}" != "-" ]; then
        CA_VERSION=$1;shift
        if [ -n  "$1" ] && [ "${1:0:1}" != "-" ]; then
            THIRDPARTY_IMAGE_VERSION=$1;shift
        fi
    fi
fi

# prior to 1.2.0 architecture was determined by uname -m
if [[ $VERSION =~ ^1\.[0-1]\.* ]]; then
    export FABRIC_TAG=${MARCH}-${VERSION}
    export CA_TAG=${MARCH}-${CA_VERSION}
    export THIRDPARTY_TAG=${MARCH}-${THIRDPARTY_IMAGE_VERSION}
else
    # starting with 1.2.0, multi-arch images will be default
    : "${CA_TAG:="$CA_VERSION"}"
    : "${FABRIC_TAG:="$VERSION"}"
    : "${THIRDPARTY_TAG:="$THIRDPARTY_IMAGE_VERSION"}"
fi

BINARY_FILE=hyperledger-fabric-${ARCH}-${VERSION}.tar.gz
CA_BINARY_FILE=hyperledger-fabric-ca-${ARCH}-${CA_VERSION}.tar.gz

# then parse opts
while getopts "h?dsb" opt; do
    case "$opt" in
        h|\?)
            printHelp
            exit 0
            ;;
        d)  DOCKER=false
            ;;
        s)  SAMPLES=false
            ;;
        b)  BINARIES=false
            ;;
    esac
done

if [ "$SAMPLES" == "true" ]; then
    echo
    echo "Clone hyperledger/fabric-samples repo"
    echo
    cloneSamplesRepo
fi
if [ "$BINARIES" == "true" ]; then
    echo
    echo "Pull Hyperledger Fabric binaries"
    echo
    pullBinaries
fi
if [ "$DOCKER" == "true" ]; then
    echo
    echo "Pull Hyperledger Fabric docker images"
    echo
    pullDockerImages
fi
```





```bash
chengchao@web1:~/gopath/src/github.com/chengchaos:) ./bootstrap.sh 

Clone hyperledger/fabric-samples repo

===> Cloning hyperledger/fabric-samples repo
Cloning into 'fabric-samples'...
remote: Enumerating objects: 9165, done.
remote: Counting objects: 100% (1/1), done.
remote: Total 9165 (delta 0), reused 1 (delta 0), pack-reused 9164
Receiving objects: 100% (9165/9165), 5.21 MiB | 8.99 MiB/s, done.
Resolving deltas: 100% (4898/4898), done.
fabric-samples v2.4.0 does not exist, defaulting to main. fabric-samples main branch is intended to work with recent versions of fabric.

Pull Hyperledger Fabric binaries

===> Downloading version 2.4.0 platform specific fabric binaries
===> Downloading:  https://github.com/hyperledger/fabric/releases/download/v2.4.0/hyperledger-fabric-linux-amd64-2.4.0.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   680  100   680    0     0   1201      0 --:--:-- --:--:-- --:--:--  1199
100 75.4M  100 75.4M    0     0  61038      0  0:21:35  0:21:35 --:--:-- 73891
==> Done.
===> Downloading version 1.5.2 platform specific fabric-ca-client binary
===> Downloading:  https://github.com/hyperledger/fabric-ca/releases/download/v1.5.2/hyperledger-fabric-ca-linux-amd64-1.5.2.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   683  100   683    0     0   1119      0 --:--:-- --:--:-- --:--:--  1119
100 25.4M  100 25.4M    0     0  39390      0  0:11:18  0:11:18 --:--:-- 54854
==> Done.

Pull Hyperledger Fabric docker images

FABRIC_IMAGES: peer orderer ccenv tools baseos
===> Pulling fabric Images
====> hyperledger/fabric-peer:2.4.0
2.4.0: Pulling from hyperledger/fabric-peer
97518928ae5f: Pull complete 
c046c0bc1de3: Pull complete 
0f4aa5ce2721: Pull complete 
69d8cff068a5: Pull complete 
2617c3f9ca10: Pull complete 
7b4592c61880: Pull complete 
Digest: sha256:f3a180c8be4502bd24f9c22c7f9fa8ac62dfde4ba06295f444addc7c446abfce
Status: Downloaded newer image for hyperledger/fabric-peer:2.4.0
docker.io/hyperledger/fabric-peer:2.4.0
====> hyperledger/fabric-orderer:2.4.0
2.4.0: Pulling from hyperledger/fabric-orderer
97518928ae5f: Already exists 
c046c0bc1de3: Already exists 
d4c24124e37b: Pull complete 
132855b30e0f: Pull complete 
296df2bbdc05: Pull complete 
575980e91300: Pull complete 
65f1e84b5a48: Pull complete 
Digest: sha256:4a0eacf9ba889e2cd972ad75c947f9049e9b5605da7defd0fc4b44749354f31b
Status: Downloaded newer image for hyperledger/fabric-orderer:2.4.0
docker.io/hyperledger/fabric-orderer:2.4.0
====> hyperledger/fabric-ccenv:2.4.0
2.4.0: Pulling from hyperledger/fabric-ccenv
a0d0a0d46f8b: Pull complete 
31adcdaf11c8: Pull complete 
b8b176561691: Pull complete 
ffa5077b735b: Pull complete 
2e51fde7a4ad: Pull complete 
10e1df71a7fd: Pull complete 
d18ada9fcb9a: Pull complete 
a3436d02b0b6: Pull complete 
b2e986580802: Pull complete 
Digest: sha256:73ea657b584f528642f32513ab448dc51fb2e099b610dd99145b5bdc5cf231bf
Status: Downloaded newer image for hyperledger/fabric-ccenv:2.4.0
docker.io/hyperledger/fabric-ccenv:2.4.0
====> hyperledger/fabric-tools:2.4.0
2.4.0: Pulling from hyperledger/fabric-tools
a0d0a0d46f8b: Already exists 
31adcdaf11c8: Already exists 
b8b176561691: Already exists 
ffa5077b735b: Already exists 
2e51fde7a4ad: Already exists 
4aa29a5074a4: Pull complete 
9b1a4931ded9: Pull complete 
7ccd225d4878: Pull complete 
Digest: sha256:82c1052cd7873723ef3c03ece7ee236031287b02cd42ba9659a48b482749d65a
Status: Downloaded newer image for hyperledger/fabric-tools:2.4.0
docker.io/hyperledger/fabric-tools:2.4.0
====> hyperledger/fabric-baseos:2.4.0
2.4.0: Pulling from hyperledger/fabric-baseos
97518928ae5f: Already exists 
c046c0bc1de3: Already exists 
a8591e840a1b: Pull complete 
Digest: sha256:30a943127742b5dfd60a0778866c6098a54270ab77a5a61618ec2182754a52ab
Status: Downloaded newer image for hyperledger/fabric-baseos:2.4.0
docker.io/hyperledger/fabric-baseos:2.4.0
===> Pulling fabric ca Image
====> hyperledger/fabric-ca:1.5.2
1.5.2: Pulling from hyperledger/fabric-ca
a0d0a0d46f8b: Already exists 
ac8258c0aeb1: Pull complete 
6c802cf1fa97: Pull complete 
Digest: sha256:faa3b743d9ed391c30f518a7cc1168160bf335f3bf60ba6aaaf1aa49c1ed023e
Status: Downloaded newer image for hyperledger/fabric-ca:1.5.2
docker.io/hyperledger/fabric-ca:1.5.2
===> List out hyperledger docker images
hyperledger/fabric-tools            2.4                 58120bdf5a41        3 weeks ago         458MB
hyperledger/fabric-tools            2.4.0               58120bdf5a41        3 weeks ago         458MB
hyperledger/fabric-tools            latest              58120bdf5a41        3 weeks ago         458MB
hyperledger/fabric-peer             2.4                 4000f61a7d44        3 weeks ago         54.8MB
hyperledger/fabric-peer             2.4.0               4000f61a7d44        3 weeks ago         54.8MB
hyperledger/fabric-peer             latest              4000f61a7d44        3 weeks ago         54.8MB
hyperledger/fabric-orderer          2.4                 1fec842b8f3e        3 weeks ago         37.2MB
hyperledger/fabric-orderer          2.4.0               1fec842b8f3e        3 weeks ago         37.2MB
hyperledger/fabric-orderer          latest              1fec842b8f3e        3 weeks ago         37.2MB
hyperledger/fabric-ccenv            2.4                 2f4d3b992cf1        3 weeks ago         504MB
hyperledger/fabric-ccenv            2.4.0               2f4d3b992cf1        3 weeks ago         504MB
hyperledger/fabric-ccenv            latest              2f4d3b992cf1        3 weeks ago         504MB
hyperledger/fabric-baseos           2.4                 2d7964efb917        3 weeks ago         6.94MB
hyperledger/fabric-baseos           2.4.0               2d7964efb917        3 weeks ago         6.94MB
hyperledger/fabric-baseos           latest              2d7964efb917        3 weeks ago         6.94MB
hyperledger/fabric-ca               1.5                 4ea287b75c63        3 months ago        69.8MB
hyperledger/fabric-ca               1.5.2               4ea287b75c63        3 months ago        69.8MB
hyperledger/fabric-ca               latest              4ea287b75c63        3 months ago        69.8MB
chengchao@web1:~/gopath/src/github.com/chengchaos:) 
```

## 运行 Fabric

- [Running a Test Network](https://hyperledger-fabric.readthedocs.io/en/latest/test_network.html) 教程 - 帮助您理解 Fabric 网络是怎样工作的
- [Running a Fabric Application](https://hyperledger-fabric.readthedocs.io/en/latest/write_first_app.html) 教程 - 帮助你在 Fabric 网络之上开发区块链应用。

Both tutorials will link to deeper explanations in [Key Concepts](https://hyperledger-fabric.readthedocs.io/en/latest/key_concepts.html).

## 使用 Fabric test network

After you have downloaded the Hyperledger Fabric Docker images and samples, you can deploy a test network by using scripts that are provided in the `fabric-samples` repository. The test network is provided for learning about Fabric by running nodes on your local machine. 

Developers can use the network to test their smart contracts and applications. The network is  meant to be used only as a tool for education and testing and not as a model for  how to set up a network. In general, modifications to the scripts are discouraged and  could break the network. It is based on a limited configuration that  should not be used as a template for deploying a production network:

- It includes two peer organizations and an ordering organization.
- For simplicity, a single node Raft ordering service is configured.
- To reduce complexity, a TLS Certificate Authority (CA) is not deployed. All certificates are issued by the root CAs.
- The sample network deploys a Fabric network with Docker Compose. Because the nodes are isolated within a Docker Compose network, the test network is not configured to connect to other running Fabric nodes.

To learn how to use Fabric in production, see [Deploying a production network](https://hyperledger-fabric.readthedocs.io/en/latest/deployment_guide_overview.html).

> **Note:** These instructions have been verified to work against the latest stable Fabric Docker images and the pre-compiled setup utilities within the supplied tar file. If you run these commands with images or tools from the current main branch, it is possible that you will encounter errors.

## Before you begin

Before you can run the test network, you need to install Fabric Samples in your environment. Follow the instructions on [getting_started](https://hyperledger-fabric.readthedocs.io/en/latest/getting_started.html) to install the required software.

**Note:** The test network has been successfully  verified with Docker Desktop version 2.5.0.1 and is the recommended  version at this time. Higher versions may not work.



## Bring up the test network

You can find the scripts to bring up the network in the `test-network` directory of the `fabric-samples` repository. Navigate to the test network directory by using the following command:

```bash
cd fabric-samples/test-network
```

In this directory, you can find an annotated script, `network.sh`, that stands up a Fabric network using the Docker images on your local machine. You can run `./network.sh -h` to print the script help text:

```bash
Usage: 
  network.sh <Mode> [Flags]
    Modes:
      up - Bring up Fabric orderer and peer nodes. No channel is created
      up createChannel - Bring up fabric network with one channel
      createChannel - Create and join a channel after the network is created
      deployCC - Deploy a chaincode to a channel (defaults to asset-transfer-basic)
      down - Bring down the network

    Flags:
    Used with network.sh up, network.sh createChannel:
    -ca <use CAs> -  Use Certificate Authorities to generate network crypto material
    -c <channel name> - Name of channel to create (defaults to "mychannel")
    -s <dbtype> - Peer state database to deploy: goleveldb (default) or couchdb
    -r <max retry> - CLI times out after certain number of attempts (defaults to 5)
    -d <delay> - CLI delays for a certain number of seconds (defaults to 3)
    -verbose - Verbose mode

    Used with network.sh deployCC
    -c <channel name> - Name of channel to deploy chaincode to
    -ccn <name> - Chaincode name.
    -ccl <language> - Programming language of the chaincode to deploy: go, java, javascript, typescript
    -ccv <version>  - Chaincode version. 1.0 (default), v2, version3.x, etc
    -ccs <sequence>  - Chaincode definition sequence. Must be an integer, 1 (default), 2, 3, etc
    -ccp <path>  - File path to the chaincode.
    -ccep <policy>  - (Optional) Chaincode endorsement policy using signature policy syntax. The default policy requires an endorsement from Org1 and Org2
    -cccg <collection-config>  - (Optional) File path to private data collections configuration file
    -cci <fcn name>  - (Optional) Name of chaincode initialization function. When a function is provided, the execution of init will be requested and the function will be invoked.

    -h - Print this message

 Possible Mode and flag combinations
   up -ca -r -d -s -verbose
   up createChannel -ca -c -r -d -s -verbose
   createChannel -c -r -d -verbose
   deployCC -ccn -ccl -ccv -ccs -ccp -cci -r -d -verbose

 Examples:
   network.sh up createChannel -ca -c mychannel -s couchdb
   network.sh createChannel -c channelName
   network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-javascript/ -ccl javascript
   network.sh deployCC -ccn mychaincode -ccp ./user/mychaincode -ccv 1 -ccl javascript
```

From inside the `test-network` directory, run the following command to remove any containers or artifacts from any previous runs:

```
./network.sh down
```

You can then bring up the network by issuing the following command. You will experience problems if you try to run the script from another directory:

```
./network.sh up
Starting nodes with CLI timeout of '5' tries and CLI delay of '3' seconds and using database 'leveldb' 
LOCAL_VERSION=2.4.0
DOCKER_IMAGE_VERSION=2.4.0
/bin/sh: /tmp/_MEItvR8gv/libreadline.so.7: no version information available (required by /bin/sh)
Creating network "fabric_test" with the default driver
Creating volume "docker_orderer.example.com" with default driver
Creating volume "docker_peer0.org1.example.com" with default driver
Creating volume "docker_peer0.org2.example.com" with default driver
Creating peer0.org1.example.com ... done
Creating orderer.example.com    ... done
Creating peer0.org2.example.com ... done
Creating cli                    ... done
CONTAINER ID        IMAGE                               COMMAND             CREATED             STATUS                  PORTS                                                                    NAMES
92cb49bcf14c        hyperledger/fabric-tools:latest     "/bin/bash"         1 second ago        Up Less than a second                                                                            cli
2b488a78ce71        hyperledger/fabric-orderer:latest   "orderer"           3 seconds ago       Up 1 second             0.0.0.0:7050->7050/tcp, 0.0.0.0:7053->7053/tcp, 0.0.0.0:9443->9443/tcp   orderer.example.com
eb857aff003e        hyperledger/fabric-peer:latest      "peer node start"   3 seconds ago       Up 1 second             0.0.0.0:9051->9051/tcp, 7051/tcp, 0.0.0.0:9445->9445/tcp                 peer0.org2.example.com
d214d73b2333        hyperledger/fabric-peer:latest      "peer node start"   3 seconds ago       Up 1 second             0.0.0.0:7051->7051/tcp, 0.0.0.0:9444->9444/tcp                           peer0.org1.example.com
```



## 安装 Docker compose

参考： [https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)

Run this command to download the current stable release of Docker Compose:

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

> To install a different version of Compose, substitute `1.29.2` with the version of Compose you want to use.

If you have problems installing with `curl`, see [Alternative Install Options](https://docs.docker.com/compose/install/#alternative-install-options) tab above.

Apply executable permissions to the binary:

```
sudo chmod +x /usr/local/bin/docker-compose
```

> **Note**: If the command `docker-compose` fails after installation, check your path. You can also create a symbolic link to `/usr/bin` or any other directory in your path.

For example:

```
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

1. Optionally, install [command completion](https://docs.docker.com/compose/completion/) for the `bash` and `zsh` shell.
2. Test the installation.

```
docker-compose --version
docker-compose version 1.29.2, build 5becea4c
```

### The components of the test network

After your test network is deployed, you can take some time to examine its components. Run the following command to list all of Docker containers that are running on your machine. You should see the three nodes that were created by the `network.sh` script:

```bash
docker ps -a
```

Each node and user that interacts with a Fabric network needs to belong to an organization in order to participate in the network. The test network includes two peer organizations, Org1 and Org2. It also includes a single orderer organization that maintains the ordering service of the network.

[Peers](https://hyperledger-fabric.readthedocs.io/en/latest/peers/peers.html) are the fundamental components of any Fabric network. Peers store the blockchain ledger and validate transactions before they are committed to the ledger. Peers run the smart contracts that contain the business logic that is used to manage the assets on the blockchain ledger.

Every peer in the network needs to belong to an organization. In the test network, each organization operates one peer each, `peer0.org1.example.com` and `peer0.org2.example.com`.

Every Fabric network also includes an [ordering service](https://hyperledger-fabric.readthedocs.io/en/latest/orderer/ordering_service.html). While peers validate transactions and add blocks of transactions to the blockchain ledger, they do not decide on the order of transactions or include them into new blocks. On a distributed network, peers may be running far away from each other and not have a common view of when a transaction was created. Coming to consensus on the order of transactions is a costly process that would create overhead for the peers.

An ordering service allows peers to focus on validating transactions and committing them to the ledger. After ordering nodes receive endorsed transactions from clients, they come to consensus on the order of transactions and then add them to blocks. The blocks are then distributed to peer nodes, which add the blocks to the blockchain ledger.

The sample network uses a single node Raft ordering service that is operated by the orderer organization. You can see the ordering node running on your machine as `orderer.example.com`. While the test network only uses a single node ordering service, a production network would have multiple ordering nodes, operated by one or multiple orderer organizations. The different ordering nodes would use the Raft consensus algorithm to come to agreement on the order of transactions across the network.

## Creating a channel

Now that we have peer and orderer nodes running on our machine, we can use the script to create a Fabric channel for transactions between Org1 and Org2. Channels are a private layer of communication between specific network members. Channels can be used only by organizations that are invited to the channel, and are invisible to other members of the network. Each channel has a separate blockchain ledger. Organizations that have been invited “join” their peers to the channel to store the channel ledger and validate the transactions on the channel.

现在我们有了 peer 和 orderer 节点运行在我们的机器上，我们可以使用脚本来创建一个在 Org1 和 Org2 之间交易的 Fabric channel 。 Channels 是一个指定网络成员间通讯的私有层。Channels 只能被邀请到 channel 的组织使用，并且也是网络中其他成员不可见的。每一个 channel 有一个分开的区块链账本。被邀请的组织“加入”到它们的 peer 的通道中，以存储 channel 的账本以及在 channel 上验证交易。

You can use the `network.sh` script to create a channel between Org1 and Org2 and join their peers to the channel. Run the following command to create a channel with the default name of `mychannel`:

我们可以使用 `network.sh` 脚本创建一个在 Org1 和 Org2 之间的的 channel 然后加入它们的 peers 。 运行下面的命令创建一个 channel ，使用默认的名称 mychannel :

```bash
./network.sh createChannel
Creating channel 'mychannel'.
If network is not up, starting nodes with CLI timeout of '5' tries and CLI delay of '3' seconds and using database 'leveldb 
Generating channel genesis block 'mychannel.block'
/home/chengchao/gopath/src/github.com/chengchaos/fabric-samples/bin/configtxgen
+ configtxgen -profile TwoOrgsApplicationGenesis -outputBlock ./channel-artifacts/mychannel.block -channelID mychannel
2021-12-27 15:16:14.565 CST 0001 INFO [common.tools.configtxgen] main -> Loading configuration
2021-12-27 15:16:14.571 CST 0002 INFO [common.tools.configtxgen.localconfig] completeInitialization -> orderer type: etcdraft
2021-12-27 15:16:14.572 CST 0003 INFO [common.tools.configtxgen.localconfig] completeInitialization -> Orderer.EtcdRaft.Options unset, setting to tick_interval:"500ms" election_tick:10 heartbeat_tick:1 max_inflight_blocks:5 snapshot_interval_size:16777216 
2021-12-27 15:16:14.572 CST 0004 INFO [common.tools.configtxgen.localconfig] Load -> Loaded configuration: /home/chengchao/gopath/src/github.com/chengchaos/fabric-samples/test-network/configtx/configtx.yaml
2021-12-27 15:16:14.573 CST 0005 INFO [common.tools.configtxgen] doOutputBlock -> Generating genesis block
2021-12-27 15:16:14.573 CST 0006 INFO [common.tools.configtxgen] doOutputBlock -> Creating application channel genesis block
2021-12-27 15:16:14.574 CST 0007 INFO [common.tools.configtxgen] doOutputBlock -> Writing genesis block
+ res=0
Creating channel mychannel
Using organization 1
+ osnadmin channel join --channelID mychannel --config-block ./channel-artifacts/mychannel.block -o localhost:7053 --ca-file /home/chengchao/gopath/src/github.com/chengchaos/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --client-cert /home/chengchao/gopath/src/github.com/chengchaos/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt --client-key /home/chengchao/gopath/src/github.com/chengchaos/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.key
+ res=0
Status: 201
{
        "name": "mychannel",
        "url": "/participation/v1/channels/mychannel",
        "consensusRelation": "consenter",
        "status": "active",
        "height": 1
}

Channel 'mychannel' created
Joining org1 peer to the channel...
Using organization 1
+ peer channel join -b ./channel-artifacts/mychannel.block
+ res=0
2021-12-27 15:16:20.742 CST 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2021-12-27 15:16:20.772 CST 0002 INFO [channelCmd] executeJoin -> Successfully submitted proposal to join channel
Joining org2 peer to the channel...
Using organization 2
+ peer channel join -b ./channel-artifacts/mychannel.block
+ res=0
2021-12-27 15:16:23.828 CST 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2021-12-27 15:16:23.863 CST 0002 INFO [channelCmd] executeJoin -> Successfully submitted proposal to join channel
Setting anchor peer for org1...
Using organization 1
Fetching channel config for channel mychannel
Using organization 1
Fetching the most recent configuration block for the channel
+ peer channel fetch config config_block.pb -o orderer.example.com:7050 --ordererTLSHostnameOverride orderer.example.com -c mychannel --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
2021-12-27 07:16:24.215 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2021-12-27 07:16:24.218 UTC 0002 INFO [cli.common] readBlock -> Received block: 0
2021-12-27 07:16:24.218 UTC 0003 INFO [channelCmd] fetch -> Retrieving last config block: 0
2021-12-27 07:16:24.219 UTC 0004 INFO [cli.common] readBlock -> Received block: 0
Decoding config block to JSON and isolating config to Org1MSPconfig.json
+ configtxlator proto_decode --input config_block.pb --type common.Block --output config_block.json
+ jq '.data.data[0].payload.data.config' config_block.json
Generating anchor peer update transaction for Org1 on channel mychannel
+ jq '.channel_group.groups.Application.groups.Org1MSP.values += {"AnchorPeers":{"mod_policy": "Admins","value":{"anchor_peers": [{"host": "peer0.org1.example.com","port": 7051}]},"version": "0"}}' Org1MSPconfig.json
+ configtxlator proto_encode --input Org1MSPconfig.json --type common.Config --output original_config.pb
+ configtxlator proto_encode --input Org1MSPmodified_config.json --type common.Config --output modified_config.pb
+ configtxlator compute_update --channel_id mychannel --original original_config.pb --updated modified_config.pb --output config_update.pb
+ configtxlator proto_decode --input config_update.pb --type common.ConfigUpdate --output config_update.json
+ jq .++ cat config_update.json

+ echo '{"payload":{"header":{"channel_header":{"channel_id":"mychannel", "type":2}},"data":{"config_update":{' '"channel_id":' '"mychannel",' '"isolated_data":' '{},' '"read_set":' '{' '"groups":' '{' '"Application":' '{' '"groups":' '{' '"Org1MSP":' '{' '"groups":' '{},' '"mod_policy":' '"",' '"policies":' '{' '"Admins":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Endorsement":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Readers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Writers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '}' '},' '"values":' '{' '"MSP":' '{' '"mod_policy":' '"",' '"value":' null, '"version":' '"0"' '}' '},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '},' '"write_set":' '{' '"groups":' '{' '"Application":' '{' '"groups":' '{' '"Org1MSP":' '{' '"groups":' '{},' '"mod_policy":' '"Admins",' '"policies":' '{' '"Admins":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Endorsement":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Readers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Writers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '}' '},' '"values":' '{' '"AnchorPeers":' '{' '"mod_policy":' '"Admins",' '"value":' '{' '"anchor_peers":' '[' '{' '"host":' '"peer0.org1.example.com",' '"port":' 7051 '}' ']' '},' '"version":' '"0"' '},' '"MSP":' '{' '"mod_policy":' '"",' '"value":' null, '"version":' '"0"' '}' '},' '"version":' '"1"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '}}}}'
+ configtxlator proto_encode --input config_update_in_envelope.json --type common.Envelope --output Org1MSPanchors.tx
2021-12-27 07:16:24.501 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2021-12-27 07:16:24.522 UTC 0002 INFO [channelCmd] update -> Successfully submitted channel update
Anchor peer set for org 'Org1MSP' on channel 'mychannel'
Setting anchor peer for org2...
Using organization 2
Fetching channel config for channel mychannel
Using organization 2
Fetching the most recent configuration block for the channel
+ peer channel fetch config config_block.pb -o orderer.example.com:7050 --ordererTLSHostnameOverride orderer.example.com -c mychannel --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem
2021-12-27 07:16:24.932 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2021-12-27 07:16:24.935 UTC 0002 INFO [cli.common] readBlock -> Received block: 1
2021-12-27 07:16:24.935 UTC 0003 INFO [channelCmd] fetch -> Retrieving last config block: 1
2021-12-27 07:16:24.937 UTC 0004 INFO [cli.common] readBlock -> Received block: 1
Decoding config block to JSON and isolating config to Org2MSPconfig.json
+ configtxlator proto_decode --input config_block.pb --type common.Block --output config_block.json
+ jq '.data.data[0].payload.data.config' config_block.json
Generating anchor peer update transaction for Org2 on channel mychannel
+ jq '.channel_group.groups.Application.groups.Org2MSP.values += {"AnchorPeers":{"mod_policy": "Admins","value":{"anchor_peers": [{"host": "peer0.org2.example.com","port": 9051}]},"version": "0"}}' Org2MSPconfig.json
+ configtxlator proto_encode --input Org2MSPconfig.json --type common.Config --output original_config.pb
+ configtxlator proto_encode --input Org2MSPmodified_config.json --type common.Config --output modified_config.pb
+ configtxlator compute_update --channel_id mychannel --original original_config.pb --updated modified_config.pb --output config_update.pb
+ configtxlator proto_decode --input config_update.pb --type common.ConfigUpdate --output config_update.json
+ jq ++ cat config_update.json
.+ echo '{"payload":{"header":{"channel_header":{"channel_id":"mychannel", "type":2}},"data":{"config_update":{' '"channel_id":' '"mychannel",' '"isolated_data":' '{},' '"read_set":' 
'{' '"groups":' '{' '"Application":' '{' '"groups":' '{' '"Org2MSP":' '{' '"groups":' '{},' '"mod_policy":' '"",' '"policies":' '{' '"Admins":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Endorsement":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Readers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Writers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '}' '},' '"values":' '{' '"MSP":' '{' '"mod_policy":' '"",' '"value":' null, '"version":' '"0"' '}' '},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '},' '"write_set":' '{' '"groups":' '{' '"Application":' '{' '"groups":' '{' '"Org2MSP":' '{' '"groups":' '{},' '"mod_policy":' '"Admins",' '"policies":' '{' '"Admins":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Endorsement":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Readers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '},' '"Writers":' '{' '"mod_policy":' '"",' '"policy":' null, '"version":' '"0"' '}' '},' '"values":' '{' '"AnchorPeers":' '{' '"mod_policy":' '"Admins",' '"value":' '{' '"anchor_peers":' '[' '{' '"host":' '"peer0.org2.example.com",' '"port":' 9051 '}' ']' '},' '"version":' '"0"' '},' '"MSP":' '{' '"mod_policy":' '"",' '"value":' null, '"version":' '"0"' '}' '},' '"version":' '"1"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '},' '"mod_policy":' '"",' '"policies":' '{},' '"values":' '{},' '"version":' '"0"' '}' '}}}}'
+ configtxlator proto_encode --input config_update_in_envelope.json --type common.Envelope --output Org2MSPanchors.tx
2021-12-27 07:16:25.184 UTC 0001 INFO [channelCmd] InitCmdFactory -> Endorser and orderer connections initialized
2021-12-27 07:16:25.205 UTC 0002 INFO [channelCmd] update -> Successfully submitted channel update
Anchor peer set for org 'Org2MSP' on channel 'mychannel'
Channel 'mychannel' joined
```

如果命令成功执行，可以看到 `Channel 'mychannel' joined` 的提示。

You can also use the channel flag to create a channel with custom name. As an example, the following command would create a channel named `channel1`:

```
./network.sh createChannel -c channel1
```

The channel flag also allows you to create multiple channels by specifying different channel names. After you create `mychannel` or `channel1`, you can use the command below to create a second channel named `channel2`:

```
./network.sh createChannel -c channel2
```

**NOTE:** Make sure the name of the channel applies the following restrictions:

- contains only lower case ASCII alphanumerics, dots ‘.’, and dashes ‘-‘
- is shorter than 250 characters
- starts with a letter

If you want to bring up the network and create a channel in a single step, you can use the `up` and `createChannel` modes together:

```
./network.sh up createChannel
```

## Starting a chaincode on the channel

After you have created a channel, you can start using [smart contracts](https://hyperledger-fabric.readthedocs.io/en/latest/smartcontract/smartcontract.html) to interact with the channel ledger. Smart contracts contain the business logic that governs assets on the blockchain ledger.智能合约包含管理区块链分类帐上资产的业务逻辑. Applications run by members of the network can invoke smart contracts to create assets on the ledger, as well as change and transfer those assets. Applications also query smart contracts to read data on the ledger.

To ensure that transactions are valid, transactions created using smart contracts typically need to be signed by multiple organizations to be committed to the channel ledger. Multiple signatures are integral（必须的） to the trust model of Fabric. Requiring multiple endorsements(背书) for a transaction prevents(阻止) one organization on a channel from tampering(tamper 干预) with the ledger on their peer or using business logic that was not agreed to. To sign a transaction, each organization needs to invoke and execute the smart contract on their peer, which then signs the output of the transaction. If the output is consistent and has been signed by enough organizations, the transaction can be committed to the ledger. The policy that specifies the set organizations on the channel that need to execute the smart contract is referred to as the endorsement policy, which is set for each chaincode as part of the chaincode definition.

In Fabric, smart contracts are deployed on the network in packages referred to as chaincode. A Chaincode is installed on the peers of an organization and then deployed to a channel, where it can then be used to endorse transactions and interact with the blockchain ledger. Before a chaincode can be deployed to a channel, the members of the channel need to agree on a chaincode definition that establishes chaincode governance. When the required number of organizations agree, the chaincode definition can be committed to the channel, and the chaincode is ready to be used.

After you have used the `network.sh` to create a channel, you can start a chaincode on the channel using the following command:

```bash
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
```

The `deployCC` subcommand will install the **asset-transfer (basic)** chaincode on `peer0.org1.example.com` and `peer0.org2.example.com` and then deploy the chaincode on the channel specified using the channel flag (or `mychannel` if no channel is specified).  If you are deploying a chaincode for the first time, the script will install the chaincode dependencies. You can use the language flag, `-l`, to install the Go, typescript or javascript versions of the chaincode. You can find the asset-transfer (basic) chaincode in the `asset-transfer-basic` folder of the `fabric-samples` directory. This folder contains sample chaincode that are provided as examples and used by tutorials to highlight Fabric features.

```bash
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
deploying chaincode on channel 'mychannel'
executing with the following
- CHANNEL_NAME: mychannel
- CC_NAME: basic
- CC_SRC_PATH: ../asset-transfer-basic/chaincode-go
- CC_SRC_LANGUAGE: go
- CC_VERSION: 1.0
- CC_SEQUENCE: 1
- CC_END_POLICY: NA
- CC_COLL_CONFIG: NA
- CC_INIT_FCN: NA
- DELAY: 3
- MAX_RETRY: 5
- VERBOSE: false
Vendoring Go dependencies at ../asset-transfer-basic/chaincode-go
~/gopath/src/github.com/chengchaos/fabric-samples/asset-transfer-basic/chaincode-go ~/gopath/src/github.com/chengchaos/fabric-samples/test-network
go: downloading github.com/golang/protobuf v1.3.2
go: downloading github.com/hyperledger/fabric-contract-api-go v1.1.0
go: downloading github.com/hyperledger/fabric-chaincode-go v0.0.0-20200424173110-d7076418f212
go: downloading github.com/hyperledger/fabric-protos-go v0.0.0-20200424173316-dd554ba3746e
go: downloading github.com/stretchr/testify v1.5.1
go: downloading github.com/xeipuuv/gojsonschema v1.2.0
go: downloading google.golang.org/grpc v1.23.0
go: downloading github.com/gobuffalo/packr v1.30.1
go: downloading github.com/xeipuuv/gojsonreference v0.0.0-20180127040603-bd5ef7bd5415
go: downloading github.com/gobuffalo/packd v0.3.0
go: downloading github.com/go-openapi/spec v0.19.4
go: downloading github.com/gobuffalo/envy v1.7.0
go: downloading github.com/go-openapi/jsonreference v0.19.2
go: downloading github.com/joho/godotenv v1.3.0
go: downloading github.com/go-openapi/jsonpointer v0.19.3
go: downloading github.com/xeipuuv/gojsonpointer v0.0.0-20180127040702-4e3ac2762d5f
go: downloading google.golang.org/genproto v0.0.0-20180831171423-11092d34479b
go: downloading golang.org/x/net v0.0.0-20190827160401-ba9fcec4b297
go: downloading github.com/rogpeppe/go-internal v1.3.0
go: downloading github.com/go-openapi/swag v0.19.5
go: downloading gopkg.in/yaml.v2 v2.2.8
go: downloading github.com/davecgh/go-spew v1.1.1
go: downloading github.com/mailru/easyjson v0.0.0-20190626092158-b2ccc519800e
go: downloading github.com/pmezard/go-difflib v1.0.0
go: downloading github.com/PuerkitoBio/purell v1.1.1
go: downloading golang.org/x/text v0.3.2
go: downloading golang.org/x/sys v0.0.0-20190710143415-6ec70d6a5542
go: downloading github.com/PuerkitoBio/urlesc v0.0.0-20170810143723-de5bf2ad4578
~/gopath/src/github.com/chengchaos/fabric-samples/test-network
Finished vendoring Go dependencies
+ peer lifecycle chaincode package basic.tar.gz --path ../asset-transfer-basic/chaincode-go --lang golang --label basic_1.0
+ res=0
Chaincode is packaged
Installing chaincode on peer0.org1...
Using organization 1
+ peer lifecycle chaincode install basic.tar.gz
+ res=0
2021-12-27 15:38:13.870 CST 0001 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Installed remotely: response:<status:200 payload:"\nJbasic_1.0:dee2d612e15f5059478b9048fa4b3c9f792096554841d642b9b59099fa0e04a4\022\tbasic_1.0" > 
2021-12-27 15:38:13.870 CST 0002 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Chaincode code package identifier: basic_1.0:dee2d612e15f5059478b9048fa4b3c9f792096554841d642b9b59099fa0e04a4
Chaincode is installed on peer0.org1
Install chaincode on peer0.org2...
Using organization 2
+ peer lifecycle chaincode install basic.tar.gz
+ res=0
2021-12-27 15:38:26.033 CST 0001 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Installed remotely: response:<status:200 payload:"\nJbasic_1.0:dee2d612e15f5059478b9048fa4b3c9f792096554841d642b9b59099fa0e04a4\022\tbasic_1.0" > 
2021-12-27 15:38:26.034 CST 0002 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Chaincode code package identifier: basic_1.0:dee2d612e15f5059478b9048fa4b3c9f792096554841d642b9b59099fa0e04a4
Chaincode is installed on peer0.org2
Using organization 1
+ peer lifecycle chaincode queryinstalled
+ res=0
Installed chaincodes on peer:
Package ID: basic_1.0:dee2d612e15f5059478b9048fa4b3c9f792096554841d642b9b59099fa0e04a4, Label: basic_1.0
Query installed successful on peer0.org1 on channel
Using organization 1
+ peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile /home/chengchao/gopath/src/github.com/chengchaos/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --channelID mychannel --name basic --version 1.0 --package-id basic_1.0:dee2d612e15f5059478b9048fa4b3c9f792096554841d642b9b59099fa0e04a4 --sequence 1
+ res=0
2021-12-27 15:38:28.170 CST 0001 INFO [chaincodeCmd] ClientWait -> txid [f9b4be2c1d61b53dbf15d7c5be230a4d62ad4333edddae641f0b44e91f0f1a6f] committed with status (VALID) at localhost:7051
Chaincode definition approved on peer0.org1 on channel 'mychannel'
Using organization 1
Checking the commit readiness of the chaincode definition on peer0.org1 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org1, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic --version 1.0 --sequence 1 --output json
+ res=0
{
        "approvals": {
                "Org1MSP": true,
                "Org2MSP": false
        }
}
Checking the commit readiness of the chaincode definition successful on peer0.org1 on channel 'mychannel'
Using organization 2
Checking the commit readiness of the chaincode definition on peer0.org2 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org2, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic --version 1.0 --sequence 1 --output json
+ res=0
{
        "approvals": {
                "Org1MSP": true,
                "Org2MSP": false
        }
}
Checking the commit readiness of the chaincode definition successful on peer0.org2 on channel 'mychannel'
Using organization 2
+ peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile /home/chengchao/gopath/src/github.com/chengchaos/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --channelID mychannel --name basic --version 1.0 --package-id basic_1.0:dee2d612e15f5059478b9048fa4b3c9f792096554841d642b9b59099fa0e04a4 --sequence 1
+ res=0
2021-12-27 15:38:36.378 CST 0001 INFO [chaincodeCmd] ClientWait -> txid [511730f97d15ce2c4465b6c868089eb00a9d5dfe7632308aec18997965ef8804] committed with status (VALID) at localhost:9051
Chaincode definition approved on peer0.org2 on channel 'mychannel'
Using organization 1
Checking the commit readiness of the chaincode definition on peer0.org1 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org1, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic --version 1.0 --sequence 1 --output json
+ res=0
{
        "approvals": {
                "Org1MSP": true,
                "Org2MSP": true
        }
}
Checking the commit readiness of the chaincode definition successful on peer0.org1 on channel 'mychannel'
Using organization 2
Checking the commit readiness of the chaincode definition on peer0.org2 on channel 'mychannel'...
Attempting to check the commit readiness of the chaincode definition on peer0.org2, Retry after 3 seconds.
+ peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic --version 1.0 --sequence 1 --output json
+ res=0
{
        "approvals": {
                "Org1MSP": true,
                "Org2MSP": true
        }
}
Checking the commit readiness of the chaincode definition successful on peer0.org2 on channel 'mychannel'
Using organization 1
Using organization 2
+ peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile /home/chengchao/gopath/src/github.com/chengchaos/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --channelID mychannel --name basic --peerAddresses localhost:7051 --tlsRootCertFiles /home/chengchao/gopath/src/github.com/chengchaos/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles /home/chengchao/gopath/src/github.com/chengchaos/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt --version 1.0 --sequence 1
+ res=0
2021-12-27 15:38:44.693 CST 0001 INFO [chaincodeCmd] ClientWait -> txid [074d6d4df733d1f144043e15beb3de123fb4c55394b3f15666f4f749eaf611b3] committed with status (VALID) at localhost:9051
2021-12-27 15:38:44.742 CST 0002 INFO [chaincodeCmd] ClientWait -> txid [074d6d4df733d1f144043e15beb3de123fb4c55394b3f15666f4f749eaf611b3] committed with status (VALID) at localhost:7051
Chaincode definition committed on channel 'mychannel'
Using organization 1
Querying chaincode definition on peer0.org1 on channel 'mychannel'...
Attempting to Query committed status on peer0.org1, Retry after 3 seconds.
+ peer lifecycle chaincode querycommitted --channelID mychannel --name basic
+ res=0
Committed chaincode definition for chaincode 'basic' on channel 'mychannel':
Version: 1.0, Sequence: 1, Endorsement Plugin: escc, Validation Plugin: vscc, Approvals: [Org1MSP: true, Org2MSP: true]
Query chaincode definition successful on peer0.org1 on channel 'mychannel'
Using organization 2
Querying chaincode definition on peer0.org2 on channel 'mychannel'...
Attempting to Query committed status on peer0.org2, Retry after 3 seconds.
+ peer lifecycle chaincode querycommitted --channelID mychannel --name basic
+ res=0
Committed chaincode definition for chaincode 'basic' on channel 'mychannel':
Version: 1.0, Sequence: 1, Endorsement Plugin: escc, Validation Plugin: vscc, Approvals: [Org1MSP: true, Org2MSP: true]
Query chaincode definition successful on peer0.org2 on channel 'mychannel'
Chaincode initialization is not required
```

## Interacting with the network

After you bring up the test network, you can use the `peer` CLI to interact with your network. The `peer` CLI allows you to invoke deployed smart contracts, update channels, or install and deploy new smart contracts from the CLI.

Make sure that you are operating from the `test-network` directory. If you followed the instructions to [install the Samples, Binaries and Docker Images](https://hyperledger-fabric.readthedocs.io/en/latest/install.html), You can find the `peer` binaries in the `bin` folder of the `fabric-samples` repository. Use the following command to add those binaries to your CLI Path:

```bash
export PATH=${PWD}/../bin:$PATH
```

You also need to set the `FABRIC_CFG_PATH` to point to the `core.yaml` file in the `fabric-samples` repository:

```
export FABRIC_CFG_PATH=$PWD/../config/
```

You can now set the environment variables that allow you to operate the `peer` CLI as Org1:

```bash
# Environment variables for Org1

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```

The `CORE_PEER_TLS_ROOTCERT_FILE` and `CORE_PEER_MSPCONFIGPATH` environment variables point to the Org1 crypto material in the `organizations` folder.

If you used `./network.sh deployCC -ccl go` to install and start the asset-transfer (basic) chaincode, you can invoke the `InitLedger` function of the (Go) chaincode to put an initial list of assets on the ledger (if using TypeScript or JavaScript `./network.sh deployCC -ccl javascript` for example, you will invoke the `InitLedger` function of the respective chaincodes).

Run the following command to initialize the ledger with assets. (Note the CLI does not access the Fabric Gateway peer, so each endorsing peer must be specified.)

```
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses localhost:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" -c '{"function":"InitLedger","Args":[]}'
2021-12-27 15:43:23.684 CST 0001 INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200 
```

If successful, you should see output similar to the following example:

```
-> INFO 001 Chaincode invoke successful. result: status:200
```

You can now query the ledger from your CLI. Run the following command to get the list of assets that were added to your channel ledger:

```
peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
```

如果成功，可以看到以下输出：

```json
[
    {
        "AppraisedValue":300,
        "Color":"blue",
        "ID":"asset1",
        "Owner":"Tomoko",
        "Size":5
    },
    {
        "AppraisedValue":400,
        "Color":"red",
        "ID":"asset2",
        "Owner":"Brad",
        "Size":5
    },
    {
        "AppraisedValue":500,
        "Color":"green",
        "ID":"asset3",
        "Owner":"Jin Soo",
        "Size":10
    },
    {
        "AppraisedValue":600,
        "Color":"yellow",
        "ID":"asset4",
        "Owner":"Max",
        "Size":10
    },
    {
        "AppraisedValue":700,
        "Color":"black",
        "ID":"asset5",
        "Owner":"Adriana",
        "Size":15
    },
    {
        "AppraisedValue":800,
        "Color":"white",
        "ID":"asset6",
        "Owner":"Michel",
        "Size":15
    }
]
```

Chaincodes are invoked when a network member wants to transfer or  change an asset on the ledger. Use the following command to change the  owner of an asset on the ledger by invoking the asset-transfer (basic)  chaincode:

```bash
peer chaincode invoke \
-o localhost:7050 \
--ordererTLSHostnameOverride orderer.example.com \
--tls \
--cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" \
-C mychannel \
-n basic \
--peerAddresses localhost:7051 \
--tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" \
--peerAddresses localhost:9051 \
--tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" \
-c '{"function":"TransferAsset","Args":["asset6","Christopher"]}'
```

If the command is successful, you should see the following response:

```bash
2021-12-27 15:59:26.166 CST 0001 INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200 payload:"Christopher" 
```

Because the endorsement policy for the asset-transfer (basic) chaincode requires the transaction to be signed by Org1 and Org2, the chaincode invoke command needs to target both `peer0.org1.example.com` and `peer0.org2.example.com` using the `--peerAddresses` flag. Because TLS is enabled for the network, the command also needs to reference the TLS certificate for each peer using the `--tlsRootCertFiles` flag.

After we invoke the chaincode, we can use another query to see how the invoke changed the assets on the blockchain ledger. Since we already queried the Org1 peer, we can take this opportunity to query the chaincode running on the Org2 peer. Set the following environment variables to operate as Org2:

```bash
# Environment variables for Org2

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```

You can now query the asset-transfer (basic) chaincode running on `peer0.org2.example.com`:

```bash
peer chaincode query -C mychannel -n basic -c '{"Args":["ReadAsset","asset6"]}'
```

The result will show that `"asset6"` was transferred to Christopher:

```json
{"ID":"asset6","color":"white","size":15,"owner":"Christopher","appraisedValue":800}
```

## Bring down the network

When you are finished using the test network, you can bring down the network with the following command:

```bash
./network.sh down
```

The command will stop and remove the node and chaincode containers, delete the organization crypto material, and remove the chaincode images from your Docker Registry. The command also removes the channel artifacts and docker volumes from previous runs, allowing you to run `./network.sh up` again if you encountered any problems.













照猫画虎的猫：

- [https://www.redhat.com/sysadmin/replacing-rclocal-systemd](https://www.redhat.com/sysadmin/replacing-rclocal-systemd)





