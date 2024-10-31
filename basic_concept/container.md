# Docker 容器

镜像（`Image`）和容器（`Container`）的关系，就像是面向对象程序设计中的 `类` 和 `实例` 一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的 [命名空间](https://en.wikipedia.org/wiki/Linux_namespaces)。因此容器可以拥有自己的 `root` 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立于宿主的系统下操作一样。这种特性使得容器封装的应用比直接在宿主运行更加安全。也因为这种隔离的特性，很多人初学 Docker 时常常会混淆容器和虚拟机。

前面讲过镜像使用的是分层存储，容器也是如此。每一个容器运行时，是以镜像为基础层，在其上创建一个当前容器的存储层，我们可以称这个为容器运行时读写而准备的存储层为 **容器存储层**。

容器存储层的生存周期和容器一样，容器消亡时，容器存储层也随之消亡。因此，任何保存于容器存储层的信息都会随容器删除而丢失。

按照 Docker 最佳实践的要求，容器不应该向其存储层内写入任何数据，容器存储层要保持无状态化。所有的文件写入操作，都应该使用 [数据卷（Volume）](../data_management/volume.md)、或者 [绑定宿主目录](../data_management/bind-mounts.md)，在这些位置的读写会跳过容器存储层，直接对宿主（或网络存储）发生读写，其性能和稳定性更高。

数据卷的生存周期独立于容器，容器消亡，数据卷不会消亡。因此，使用数据卷后，容器删除或者重新运行之后，数据却不会丢失。


-----------------------------


Docker Containers
The relationship between an image (Image) and a container (Container) is similar to the relationship between a class and an instance in object-oriented programming. An image is a static definition, while a container is a runtime instance of that image. Containers can be created, started, stopped, deleted, and paused, among other actions.

Essentially, a container is a process, but unlike a process that runs directly on the host, a containerized process runs within its own isolated namespace. This allows the container to have its own root filesystem, network configuration, process space, and even user ID space. Processes within a container operate in an isolated environment, giving the user an experience similar to working on a system separate from the host. This isolation makes applications within containers more secure than running directly on the host. Because of this isolation, many beginners often confuse containers with virtual machines when first learning Docker.

As discussed earlier, Docker images use a layered storage model, and containers follow the same principle. Each running container is based on an image as the base layer, on top of which a storage layer is created specifically for the current container. This layer, prepared for the container's runtime read and write operations, is called the container storage layer.

The lifecycle of the container storage layer matches that of the container itself: when a container is deleted, its storage layer is also removed. Therefore, any data saved in the container's storage layer will be lost upon container deletion.

According to Docker best practices, containers should not write data to their storage layer; instead, the container storage layer should remain stateless. All file write operations should use either Volumes or bind mounts, which bypass the container storage layer to interact directly with the host (or network storage), improving performance and stability.

The lifecycle of a volume is independent of the container. When a container is deleted, the volume persists, so data is retained even if the container is removed or restarted.


-----------------------------



# حاويات Docker

العلاقة بين الصورة (`Image`) والحاوية (`Container`) تشبه العلاقة بين `الفئة` و`المثال` في البرمجة كائنية التوجه؛ فالصورة هي تعريف ثابت، بينما الحاوية هي كيان يتم إنشاؤه من الصورة عند التشغيل. يمكن إنشاء الحاويات، وبدء تشغيلها، وإيقافها، وحذفها، وتعليقها وغيرها من العمليات.

الحاوية في جوهرها هي عملية، ولكنها تختلف عن العملية التي تعمل مباشرة على النظام المضيف، حيث تعمل العملية في الحاوية ضمن [مساحة أسماء](https://en.wikipedia.org/wiki/Linux_namespaces) مستقلة. وهذا يسمح للحاوية بامتلاك نظام ملفات `root` خاص بها، وإعدادات شبكة مستقلة، ومساحة عمليات خاصة، بل وحتى مساحة معرفات مستخدم خاصة. العمليات داخل الحاوية تعمل في بيئة معزولة، مما يمنح تجربة أشبه بالعمل على نظام منفصل عن النظام المضيف. هذه العزلة تجعل التطبيقات داخل الحاويات أكثر أمانًا من تشغيلها مباشرة على المضيف. وبسبب هذه العزلة، قد يختلط الأمر على الكثيرين ممن يتعلمون Docker لأول مرة، فيظنون أن الحاويات تشبه الآلات الافتراضية.

كما تمت الإشارة سابقًا، فإن Docker يعتمد على نموذج تخزين طبقي، وينطبق هذا أيضًا على الحاويات. كل حاوية تعمل باستخدام الصورة كطبقة أساسية، ويتم إنشاء طبقة تخزين فوقها خاصة بالحاوية الحالية. هذه الطبقة، التي تُعد للقراءة والكتابة أثناء التشغيل، تسمى **طبقة تخزين الحاوية**.

يتماثل عمر طبقة تخزين الحاوية مع عمر الحاوية نفسها؛ فعندما تنتهي الحاوية، يتم حذف طبقة التخزين أيضًا. ولذلك، أي بيانات تُحفظ في طبقة تخزين الحاوية ستُفقد عند حذف الحاوية.

وفقًا لأفضل ممارسات Docker، يجب ألا تقوم الحاوية بكتابة البيانات إلى طبقة التخزين الخاصة بها؛ بل يُفضل أن تظل طبقة تخزين الحاوية بدون حالة ثابتة. جميع عمليات الكتابة يجب أن تستخدم إما [وحدات التخزين (Volumes)](../data_management/volume.md) أو [الربط المباشر لمجلد المضيف](../data_management/bind-mounts.md)، حيث يتم تجاوز طبقة تخزين الحاوية لتتم عمليات القراءة والكتابة مباشرة على المضيف (أو التخزين الشبكي)، مما يحسن من الأداء والاستقرار.

يكون عمر وحدة التخزين مستقلًا عن الحاوية. فعند حذف الحاوية، تبقى وحدة التخزين ولا تُحذف؛ مما يعني أن البيانات تظل موجودة حتى في حال حذف الحاوية أو إعادة تشغيلها.
