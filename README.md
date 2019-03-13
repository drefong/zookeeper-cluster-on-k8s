# zookeeper-cluster-in-k8s
configmap：
    
configmap实现配置文件的构建，包括zoo.cfg和log4j.properties两个配置文件和脚本zkOk.sh，其中zoo.cfg是主配置文件，定义dataDir、dataLogDir、server list等。其中server list中由于pod位于同一namespace，使用简短域名解析pod_name.service_name，完整域名格式pod_name.server_name.namespace.svc.cluster.local。本范例中可以使用zk-cluster-0.zk-cluster.default.svc.cluster.local。
statefulset配置文件主要实现pod的定义，默认情况下statefulset是按照顺序、优雅的部署和扩容，这里需要并行的启动，避免三个节点启动时间差别太大，出现连接超时的情况，设置参数spec.podManagementPolicy为Parallel。这里空间申请为100M，用于datadir和datalogdir，可以根据实际情况进行调整
通过PodDisruptionBudget资源类型控制集群最小实例为2.
创建headless service，每个节点基于service有独立的域名。
基于dockerfile，构建3.4.10版本的zookeeper，在官方的基础上，去掉了gpg验证、volumn等，修改了docker-entrypoint.sh。其中docker-entrypoint.sh主要在之前的基础上，增加了myid的创建和id写入。