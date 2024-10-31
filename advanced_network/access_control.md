# 容器访问控制

容器的访问控制，主要通过 Linux 上的 `iptables` 防火墙来进行管理和实现。`iptables` 是 Linux 上默认的防火墙软件，在大部分发行版中都自带。

## 容器访问外部网络
容器要想访问外部网络，需要本地系统的转发支持。在Linux 系统中，检查转发是否打开。

```bash
$sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
```
如果为 0，说明没有开启转发，则需要手动打开。
```bash
$sysctl -w net.ipv4.ip_forward=1
```
如果在启动 Docker 服务的时候设定 `--ip-forward=true`, Docker 就会自动设定系统的 `ip_forward` 参数为 1。

## 容器之间访问
容器之间相互访问，需要两方面的支持。
* 容器的网络拓扑是否已经互联。默认情况下，所有容器都会被连接到 `docker0` 网桥上。
* 本地系统的防火墙软件 -- `iptables` 是否允许通过。

### 访问所有端口
当启动 Docker 服务（即 dockerd）的时候，默认会添加一条转发策略到本地主机 iptables 的 FORWARD 链上。策略为通过（`ACCEPT`）还是禁止（`DROP`）取决于配置`--icc=true`（缺省值）还是 `--icc=false`。当然，如果手动指定 `--iptables=false` 则不会添加 `iptables` 规则。

可见，默认情况下，不同容器之间是允许网络互通的。如果为了安全考虑，可以在 `/etc/docker/daemon.json` 文件中配置 `{"icc": false}` 来禁止它。

### 访问指定端口
在通过 `-icc=false` 关闭网络访问后，还可以通过 `--link=CONTAINER_NAME:ALIAS` 选项来访问容器的开放端口。

例如，在启动 Docker 服务时，可以同时使用 `icc=false --iptables=true` 参数来关闭允许相互的网络访问，并让 Docker 可以修改系统中的 `iptables` 规则。

此时，系统中的 `iptables` 规则可能是类似
```bash
$ sudo iptables -nL
...
Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
DROP       all  --  0.0.0.0/0            0.0.0.0/0
...
```

之后，启动容器（`docker run`）时使用 `--link=CONTAINER_NAME:ALIAS` 选项。Docker 会在 `iptable` 中为 两个容器分别添加一条 `ACCEPT` 规则，允许相互访问开放的端口（取决于 `Dockerfile` 中的 `EXPOSE` 指令）。

当添加了 `--link=CONTAINER_NAME:ALIAS` 选项后，添加了 `iptables` 规则。
```bash
$ sudo iptables -nL
...
Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
ACCEPT     tcp  --  172.17.0.2           172.17.0.3           tcp spt:80
ACCEPT     tcp  --  172.17.0.3           172.17.0.2           tcp dpt:80
DROP       all  --  0.0.0.0/0            0.0.0.0/0
```

注意：`--link=CONTAINER_NAME:ALIAS` 中的 `CONTAINER_NAME` 目前必须是 Docker 分配的名字，或使用 `--name` 参数指定的名字。主机名则不会被识别。


----------------------------------------------



# Container Access Control

Container access control is primarily managed and implemented using the `iptables` firewall on Linux. `iptables` is the default firewall software on Linux and comes pre-installed on most distributions.

## Container Access to External Networks
For containers to access external networks, the local system needs forwarding support. On Linux systems, check if forwarding is enabled.

```bash
$ sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
```
If it’s set to 0, forwarding is disabled and needs to be enabled manually.
```bash
$ sysctl -w net.ipv4.ip_forward=1
```
If you set `--ip-forward=true` when starting the Docker service, Docker will automatically set the system’s `ip_forward` parameter to 1.

## Inter-Container Access
Access between containers requires two things:
* The network topology of the containers is interconnected. By default, all containers are connected to the `docker0` bridge.
* The local system’s firewall software, `iptables`, allows traffic through.

### Accessing All Ports
When starting the Docker service (`dockerd`), a forwarding rule is added to the FORWARD chain in the local `iptables` by default. Whether the policy is set to allow (`ACCEPT`) or deny (`DROP`) depends on the `--icc=true` (default) or `--icc=false` setting. If `--iptables=false` is specified, no `iptables` rules are added.

By default, network communication between different containers is permitted. For security reasons, you can disable it by setting `{"icc": false}` in the `/etc/docker/daemon.json` file.

### Accessing Specific Ports
After disabling network access with `-icc=false`, you can still access open ports by using the `--link=CONTAINER_NAME:ALIAS` option.

For example, when starting the Docker service, you can use both `icc=false` and `--iptables=true` parameters to disable inter-container network access while allowing Docker to modify the system’s `iptables` rules.

In this case, the system’s `iptables` rules might look like this:
```bash
$ sudo iptables -nL
...
Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
DROP       all  --  0.0.0.0/0            0.0.0.0/0
...
```

Then, when starting a container (`docker run`) with the `--link=CONTAINER_NAME:ALIAS` option, Docker will add an `ACCEPT` rule to `iptables` for both containers, allowing them to access each other's open ports (according to the `EXPOSE` directive in the `Dockerfile`).

When using the `--link=CONTAINER_NAME:ALIAS` option, the `iptables` rules will be updated:
```bash
$ sudo iptables -nL
...
Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
ACCEPT     tcp  --  172.17.0.2           172.17.0.3           tcp spt:80
ACCEPT     tcp  --  172.17.0.3           172.17.0.2           tcp dpt:80
DROP       all  --  0.0.0.0/0            0.0.0.0/0
```

Note: In `--link=CONTAINER_NAME:ALIAS`, `CONTAINER_NAME` must currently be the name assigned by Docker or a name specified using the `--name` option. Hostnames will not be recognized.


----------------------------------------------



# التحكم في وصول الحاويات

يتم إدارة التحكم في وصول الحاويات بشكل أساسي من خلال جدار الحماية `iptables` على نظام Linux. يعتبر `iptables` جدار الحماية الافتراضي في Linux ويأتي مثبتاً مسبقاً في معظم التوزيعات.

## وصول الحاوية إلى الشبكات الخارجية
لكي تتمكن الحاويات من الوصول إلى الشبكات الخارجية، يجب أن يدعم النظام المحلي عملية التوجيه. في نظام Linux، يمكن التحقق من تمكين التوجيه على النحو التالي:

```bash
$ sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 1
```
إذا كانت القيمة 0، فهذا يعني أن التوجيه معطل ويحتاج إلى التمكين يدوياً:
```bash
$ sysctl -w net.ipv4.ip_forward=1
```
عند ضبط `--ip-forward=true` أثناء تشغيل خدمة Docker، سيتم ضبط النظام تلقائياً على تفعيل التوجيه (`ip_forward` = 1).

## الوصول بين الحاويات
يستلزم الوصول بين الحاويات جانبين:
* أن تكون البنية التحتية للشبكة بين الحاويات مترابطة. افتراضياً، يتم ربط جميع الحاويات بجسر `docker0`.
* أن يسمح جدار الحماية في النظام المحلي - `iptables` - بمرور البيانات.

### الوصول إلى جميع المنافذ
عند تشغيل خدمة Docker (`dockerd`)، يتم إضافة قاعدة توجيه إلى سلسلة FORWARD في `iptables` بالنظام المحلي بشكل افتراضي. يتم ضبط السياسة على القبول (`ACCEPT`) أو الرفض (`DROP`) بناءً على إعداد `--icc=true` (افتراضي) أو `--icc=false`. إذا تم تحديد `--iptables=false`، فلن يتم إضافة قواعد `iptables`.

بشكل افتراضي، يُسمح بالتواصل بين الحاويات المختلفة. لأسباب أمنية، يمكنك تعطيل ذلك بإضافة `{"icc": false}` في ملف `/etc/docker/daemon.json`.

### الوصول إلى منافذ محددة
بعد إيقاف الوصول الشبكي باستخدام `-icc=false`، يمكن الوصول إلى المنافذ المفتوحة باستخدام الخيار `--link=CONTAINER_NAME:ALIAS`.

على سبيل المثال، عند تشغيل خدمة Docker، يمكن استخدام المعاملين `icc=false` و`--iptables=true` معاً لتعطيل الوصول الشبكي بين الحاويات والسماح لـ Docker بتعديل قواعد `iptables` في النظام.

في هذه الحالة، قد تظهر قواعد `iptables` في النظام كما يلي:
```bash
$ sudo iptables -nL
...
Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
DROP       all  --  0.0.0.0/0            0.0.0.0/0
...
```

بعد ذلك، عند تشغيل الحاوية (`docker run`) باستخدام الخيار `--link=CONTAINER_NAME:ALIAS`، سيضيف Docker قاعدة `ACCEPT` في `iptables` للحاويتين، مما يسمح لهما بالوصول إلى المنافذ المفتوحة لبعضهما البعض (حسب تعليمات `EXPOSE` في `Dockerfile`).

عند إضافة الخيار `--link=CONTAINER_NAME:ALIAS`، سيتم تحديث قواعد `iptables` على النحو التالي:
```bash
$ sudo iptables -nL
...
Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
ACCEPT     tcp  --  172.17.0.2           172.17.0.3           tcp spt:80
ACCEPT     tcp  --  172.17.0.3           172.17.0.2           tcp dpt:80
DROP       all  --  0.0.0.0/0            0.0.0.0/0
```

ملاحظة: في `--link=CONTAINER_NAME:ALIAS` يجب أن يكون `CONTAINER_NAME` هو الاسم الذي تم تخصيصه بواسطة Docker أو الاسم المحدد باستخدام الخيار `--name`. أسماء المضيف لن يتم التعرف عليها.
