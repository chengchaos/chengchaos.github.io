---
title: Setting Up a Proxy for Git
key: 2026-01-16
tags: Git Proxy
---

When working with Git, you might encounter slow connection speeds, especially when cloning repositories from GitHub. Setting up a proxy can help improve the connection speed. Here are the steps to configure HTTP, HTTPS, and SOCKS5 proxies for Git.

## 0x01 Configuring HTTP and HTTPS Proxies

To set up an HTTP or HTTPS proxy, you can use the following commands:

```sh
# Set HTTP proxy
git config --global http.proxy "http://127.0.0.1:8080"

# Set HTTPS proxy
git config --global https.proxy "http://127.0.0.1:8080"
```

For SOCKS5 proxy, use the following commands:

```sh
# Set SOCKS5 proxy for HTTP
git config --global http.proxy "socks5://127.0.0.1:1080"

# Set SOCKS5 proxy for HTTPS
git config --global https.proxy "socks5://127.0.0.1:1080"
```

To remove the proxy settings, use:

```sh
# Unset HTTP proxy
git config --global --unset http.proxy

# Unset HTTPS proxy
git config --global --unset https.proxy

```

## 0x02 Configuring SSH Proxy

For SSH connections, you need to modify the `~/.ssh/config` file. If the file does not exist, create it. Add the following configuration:

```sh
# For GitHub SSH connections
Host github.com
HostName github.com
User git
# Use HTTP proxy
# ProxyCommand socat - PROXY:127.0.0.1:%h:%p,proxyport=8080
# Use SOCKS5 proxy
# ProxyCommand nc -v -x 127.0.0.1:1080 %h %p
```

For Windows users who do not have nc, you can use connect:

```sh
# Use SOCKS5 proxy with connect
ProxyCommand connect -S 127.0.0.1:1080 %h %p
```

## 0x03 One-Click Proxy Switch Script for Linux

You can create a bash script to quickly enable or disable the proxy settings. Add the following functions to your `~/.bash_profile`:

```sh
# Enable GitHub proxy
function github_on() {
cat "${HOME}/.ssh/config" > "${HOME}/.ssh/config_back" # Backup
config=$(cat ${HOME}/.ssh/config_back | sed 's/# ProxyCommand/ProxyCommand/g')
echo -e "${config}" > "${HOME}/.ssh/config" && echo -e "\033[36mGitHub proxy enabled!\033[0m"
}

# Disable GitHub proxy
function github_off() {
cat "${HOME}/.ssh/config" > "${HOME}/.ssh/config_back" # Backup
config=$(cat ${HOME}/.ssh/config_back | sed 's/ ProxyCommand/ # ProxyCommand/g')
echo -e "${config}" > "${HOME}/.ssh/config" && echo -e "\033[33mGitHub proxy disabled!\033[0m"
}
```

After adding these functions, run source ~/.bash_profile to apply the changes. You can then use the following commands to enable or disable the proxy:

```sh
# Enable proxy
github_on

# Disable proxy
github_off
```

These steps should help you set up and manage proxies for Git efficiently.