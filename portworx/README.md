## Portworx


### 存储类型支持

同时支持Block和FS，不直接支持Object对象存储，但是可以通过[Minio](https://minio.io/)来间接支持。

参考：
- https://docs.portworx.com/knowledgebase/faqs.html#do-you-support-block--file--object
- https://docs.portworx.com/applications/object-storage.html

### CSI的支持情况

目前CSI还没有完全GA，K8S v1.11中还是Beta，预计GA会在v1.12中。考虑到这一点Portworx的CSI支持目前只提供Tech Preview版本。

当前CSI支持是开源的，并且只兼容到了CSI v0.2版本，而K8S v1.11已经支持v0.3版本，不过相信Portworx很快就会支持。

参考：
- https://docs.portworx.com/scheduler/kubernetes/csi.html
- https://github.com/libopenstorage/openstorage/tree/master/csi
