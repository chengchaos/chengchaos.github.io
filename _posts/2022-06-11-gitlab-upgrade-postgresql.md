---
title: gitlab upgrade postgresql
key: 2020-05-10
tags: linux gitlab postgresql
---

公司 gitlab 使用的 PostgreSQL 版本过低, 必须升级.

<!--more-->

公司的 gitlab 不是我装的, 到我这里的时候, 什么文档都没有, 于是上网上去找方案.

第一个是:

[Using the PostgreSQL Database Service shipped with Omnibus GitLab](https://docs.gitlab.com/omnibus/settings/database.html#upgrade-packaged-postgresql-server)

但是单位使用的是 gitlab 和数据库分离部署的

[Using a non-packaged PostgreSQL database management server](https://docs.gitlab.com/omnibus/settings/database.html#using-a-non-packaged-postgresql-database-management-server)

然后, 我先去 gitlab 的服务器上更新了一下 yum

```sh
Total download size: 1.0 G
Is this ok [y/d/N]: y
Downloading packages:
Delta RPMs disabled because /usr/bin/applydeltarpm not installed.
gitlab-ce-15.0.2-ce.0.el7.x86_64.rpm                                           | 1.0 GB  00:01:23
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
Malformed configuration JSON file found at /opt/gitlab/embedded/nodes/AZCLPSVN.json.
This usually happens when your last run of `gitlab-ctl reconfigure` didn't complete successfully.
This file is used to check if any of the unsupported configurations are enabled,
and hence require a working reconfigure before upgrading.
Please run `sudo gitlab-ctl reconfigure` to fix it and try again.
error: %pre(gitlab-ce-15.0.2-ce.0.el7.x86_64) scriptlet failed, exit status 1
Error in PREIN scriptlet in rpm package gitlab-ce-15.0.2-ce.0.el7.x86_64
gitlab-ce-14.10.0-ce.0.el7.x86_64 was supposed to be removed but is not!
  Verifying  : gitlab-ce-14.10.0-ce.0.el7.x86_64                                                  1/2
  Verifying  : gitlab-ce-15.0.2-ce.0.el7.x86_64                                                   2/2

Failed:
  gitlab-ce.x86_64 0:14.10.0-ce.0.el7                gitlab-ce.x86_64 0:15.0.2-ce.0.el7

Complete!

```

执行了 `gitlab-ctl reconfigure` 出现了错误提示:

```txt


    ================================================================================
    Error executing action `run` on resource 'rails_migration[gitlab-rails]'
    ================================================================================

    Mixlib::ShellOut::ShellCommandFailed
    ------------------------------------
    bash[migrate gitlab-rails database] (/opt/gitlab/embedded/cookbooks/cache/cookbooks/gitlab/resources/rails_migration.rb line 16) had an error: Mixlib::ShellOut::ShellCommandFailed: Command execution failed. STDOUT/STDERR suppressed for sensitive resource

    Resource Declaration:
    ---------------------
    # In /opt/gitlab/embedded/cookbooks/cache/cookbooks/gitlab/recipes/database_migrations.rb

     51: rails_migration "gitlab-rails" do
     52:   rake_task 'gitlab:db:configure'
     53:   logfile_prefix 'gitlab-rails-db-migrate'
     54:   helper migration_helper
     55:
     56:   environment env_variables
     57:   dependent_services dependent_services
     58:   notifies :run, "execute[clear the gitlab-rails cache]", :immediately
     59:   notifies :run, "ruby_block[check remote PG version]", :immediately
     60:
     61:   only_if { migration_helper.attributes_node['auto_migrate'] }
     62: end

    Compiled Resource:
    ------------------
    # Declared in /opt/gitlab/embedded/cookbooks/cache/cookbooks/gitlab/recipes/database_migrations.rb:51:in `from_file'

    rails_migration("gitlab-rails") do
      action [:run]
      default_guard_interpreter :default
      declared_type :rails_migration
      cookbook_name "gitlab"
      recipe_name "database_migrations"
      rake_task "gitlab:db:configure"
      logfile_prefix "gitlab-rails-db-migrate"
      helper "*sensitive value suppressed*"
      environment "*sensitive value suppressed*"
      dependent_services ["runit_service[puma]", "sidekiq_service[sidekiq]"]
      only_if { #code block }
    end

    System Info:
    ------------
    chef_version=15.17.4
    platform=centos
    platform_version=7.9.2009
    ruby=ruby 2.7.5p203 (2021-11-24 revision f69aeb8314) [x86_64-linux]
    program_name=/opt/gitlab/embedded/bin/chef-client
    executable=/opt/gitlab/embedded/bin/chef-client


Running handlers:
There was an error running gitlab-ctl reconfigure:

rails_migration[gitlab-rails] (gitlab::database_migrations line 51) had an error: Mixlib::ShellOut::ShellCommandFailed: bash[migrate gitlab-rails database] (/opt/gitlab/embedded/cookbooks/cache/cookbooks/gitlab/resources/rails_migration.rb line 16) had an error: Mixlib::ShellOut::ShellCommandFailed: Command execution failed. STDOUT/STDERR suppressed for sensitive resource

Running handlers complete
Chef Infra Client failed. 0 resources updated in 01 minutes 34 seconds

```

参考这篇文章: [Gitlab升级14.10.0后运行出错](https://itlanyan.com/gitlab-reconfigure-error-after-upgrade-14-10-0/)

下面是照猫画的虎.

```sh
# gitlab-rake db:migrate
...中间输出一堆...

Finalize it manualy by running

        sudo gitlab-rake gitlab:background_migrations:finalize[ProjectNamespaces::BackfillProjectNamespaces,projects,id,'[null\,"up"]']

For more information, check the documentation
...中间输出一堆...

sudo gitlab-rake gitlab:background_migrations:finalize[ProjectNamespaces::BackfillProjectNamespaces,projects,id,'[null\,"up"]']
Done.
# gitlab-ctl reconfigure
# yum update
...中间输出一堆...
     _______ __  __          __
    / ____(_) /_/ /   ____ _/ /_
   / / __/ / __/ /   / __ `/ __ \
  / /_/ / / /_/ /___/ /_/ / /_/ /
  \____/_/\__/_____/\__,_/_.___/


Upgrade complete! If your GitLab server is misbehaving try running
  sudo gitlab-ctl restart
before anything else.
If you need to roll back to the previous version you can use the database
backup made during the upgrade (scroll up for the filename).

```

EOF

---

Power by TeXt.

<iframe src="https://ghbtns.com/github-btn.html?user=kitian616&repo=jekyll-TeXt-theme&type=star&count=true" frameborder="0" scrolling="0" width="170px" height="20px"></iframe>
