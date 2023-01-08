## FileScale: Fast and Elastic Metadata Management for Distributed File Systems

Distributed database systems have been shown to be effective for scaling metadata management in scalable file systems. These systems can handle billions of files, while traditional systems that store metadata on a single machine or shared-disk abstraction struggle to scale. However, distributed database systems perform worse than single-machine systems at low scales, where metadata can fit in memory. 

FileScale is a three-tier architecture that addresses this issue by incorporating a distributed database system while maintaining comparable performance to single-machine systems at small scales and enabling linear scalability as metadata increases.



## License

FileScale resources in this repository are released under the Apache License 2.0.
