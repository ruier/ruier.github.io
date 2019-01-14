---
title: Docker数据管理
categories: [blog, docker]
description: 记录的是随心所欲
keywords: docker, work
---


# [Docker数据管理(数据卷&数据卷容器)](http://www.cnblogs.com/wade-luffy/p/6542539.html)

转载：https://www.cnblogs.com/wade-luffy/p/6542539.html

生产环境中使用Docker的过程中，往往需要对数据进行持久化，或者需要在多个容器之间进行数据共享，这必然涉及容器的数据管理操作。

容器中管理数据主要有两种方式：

1. 数据卷（Data Volumes）：容器内数据直接映射到本地主机环境；如何在容器内创建数据卷，并且把本地的目录或文件挂载到容器内的数据卷中。
2. 数据卷容器（Data Volume Containers）：使用特定容器维护数据卷。如何使用数据卷容器在容器和主机、容器和容器之间共享数据，并实现数据的备份和恢复。

## 数据卷

数据卷是一个可供容器使用的特殊目录，它将主机操作系统目录直接映射进容器，类似于Linux中的mount操作。

数据卷可以提供很多有用的特性，如下所示：

1. 数据卷可以在容器之间共享和重用，容器间传递数据将变得高效方便；
2. 对数据卷内数据的修改会立马生效，无论是容器内操作还是本地操作；
3. 对数据卷的更新不会影响镜像，解耦了应用和数据；
4. 卷会一直存在，直到没有容器使用，可以安全地卸载它。

### **1.在容器内创建一个数据卷**

在用docker run命令的时候，使用-v标记可以在容器内创建一个数据卷。多次重复使用-v标记可以创建多个数据卷。

下面使用training/webapp镜像创建一个web容器，并创建一个数据卷挂载到容器的/webapp目录：

$ docker run -d -P --name web -v /webapp training/webapp python app.py

-P是将容器服务暴露的端口，是自动映射到本地主机的临时端口。

### **2.挂载一个主机目录作为数据卷**

使用-v标记也可以指定挂载一个本地的已有目录到容器中去作为数据卷（推荐方式）。

$ docker run -d -P --name web -v /src/webapp:/opt/webapp training/webapp python app.py

上面的命令加载主机的/src/webapp目录到容器的/opt/webapp目录。

这个功能在进行测试的时候十分方便，比如用户可以将一些程序或数据放到本地目录中，然后在容器内运行和使用。另外，本地目录的路径必须是绝对路径，如果目录不存在,Docker会自动创建。

Docker挂载数据卷的默认权限是读写（rw），用户也可以通过ro指定为只读：

$ docker run -d -P --name web -v /src/webapp:/opt/webapp:ro training/webapp python app.py

加了:ro之后，容器内对所挂载数据卷内的数据就无法修改了。

### **3.挂载一个本地主机文件作为数据卷**

-v标记也可以从主机挂载单个文件到容器中作为数据卷（不推荐）。

$ docker run --rm -it -v ~/.bash_history:/.bash_history ubuntu /bin/bash

这样就可以记录在容器输入过的命令历史了。

如果直接挂载一个文件到容器，使用文件编辑工具，包括vi或者sed--in-place的时候，可能会造成文件inode的改变，从Docker 1.1.0起，这会导致报错误信息。所以推荐的方式是直接挂载文件所在的目录。

## 数据卷容器

如果用户需要在多个容器之间共享一些持续更新的数据，最简单的方式是使用数据卷容器。数据卷容器也是一个容器，但是它的目的是专门用来提供数据卷供其他容器挂载。

首先，创建一个数据卷容器dbdata，并在其中创建一个数据卷挂载到/dbdata：

$ docker run -it -v /dbdata --name dbdata ubuntu

root@3ed94f279b6f:/#

查看/dbdata目录：

root@3ed94f279b6f:/# ls

bin  boot  dbdata  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run sbin  srv  sys  tmp  usr  var

然后，可以在其他容器中使用--volumes-from来挂载dbdata容器中的数据卷.

例如创建db1和db2两个容器，并从dbdata容器挂载数据卷：

$ docker run -it --volumes-from dbdata --name db1 ubuntu

$ docker run -it --volumes-from dbdata --name db2 ubuntu

此时，容器db1和db2都挂载同一个数据卷到相同的/dbdata目录。三个容器任何一方在该目录下的写入，其他容器都可以看到。

例如，在dbdata容器中创建一个test文件，如下所示：

root@3ed94f279b6f:/# cd /dbdata

root@3ed94f279b6f:/dbdata# touch test

root@3ed94f279b6f:/dbdata# ls

test

在db1容器内查看它：

$ docker run -it --volumes-from dbdata --name db1  ubuntu

root@4128d2d804b4:/# ls

bin  boot  dbdata  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

root@4128d2d804b4:/# ls dbdata/

test

可以多次使用--volumes-from参数来从多个容器挂载多个数据卷。还可以从其他已经挂载了容器卷的容器来挂载数据卷。

使用--volumes-from参数所挂载数据卷的容器自身并不需要保持在运行状态。

如果删除了挂载的容器（包括dbdata、db1和db2），数据卷并不会被自动删除。如果要删除一个数据卷，必须在删除最后一个还挂载着它的容器时显式使用docker rm -v命令来指定同时删除关联的容器。

## **利用数据卷容器来迁移数据**

可以利用数据卷容器对其中的数据卷进行备份、恢复，以实现数据的迁移。

下面介绍这两个操作。

### **1.备份**

使用下面的命令来备份dbdata数据卷容器内的数据卷：

$ docker run --volumes-from dbdata -v $(pwd):/backup --name worker ubuntu tar cvf /backup/backup.tar /dbdata

首先利用ubuntu镜像创建了一个容器worker。使用--volumes-from dbdata参数来让worker容器挂载dbdata容器的数据卷(即dbdata数据卷),使用-v  $(pwd):/backup参数来挂载本地的当前目录到worker容器的/backup目录。worker容器启动后，使用了tar cvf  /backup/backup.tar /dbdata命令来将/dbdata下内容备份为容器内的/backup/backup.tar，即宿主主机当前目录下的backup.tar。

### **2.恢复**

如果要将数据恢复到一个容器，可以按照下面的步骤操作。

首先创建一个带有数据卷的容器dbdata2：

$ docker run -v /dbdata --name dbdata2 ubuntu /bin/bash

然后创建另一个新的容器，挂载dbdata2的容器，并使用untar解压备份文件到所挂载的容器卷中：

$ docker run --volumes-from dbdata2 -v $(pwd):/backup --name worker ubuntu bash

cd /dbdata

tar xvf /backup/backup.tar
