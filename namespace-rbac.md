## Namespace
- Namespaces are Kubernetes objects which partition a single Kubernetes cluster into multiple virtual clusters. Namespaces allow you to group objects together so you can filter and control them as a unit.
1. **Isolation**. Large or growing teams can use namespaces to isolate their projects and microservices from each other. Teams can re-use the same resource names in different workspaces without a problem. Also, taking an action on items in one workspace never affects other workspaces.
1. **Organization**. Organizations that use a single cluster for development, testing, and production can use namespaces to sandbox dev and test environments. This ensures production code is not affected by changes that developers or testers make in their own namespaces throughout the application lifecycle. 
1. **Permissions**. Namespaces enable the use of Kubernetes RBAC, so teams can define roles that group lists of permissions or abilities under a single name. This can ensure that only authorized users have access to resources in a given namespace. 
1. **Resource Control**. Policy-driven resource limits can be set on namespaces by defining resource quotas for CPU or memory utilization. This can ensure that every project or namespace has the resources it needs to run, and that no one namespace is hogging all available resources.
1. **Performance**. Using namespaces can help improve performance of a given cluster. If a cluster is separated into multiple namespaces for different projects, the Kubernetes API will have fewer items to search when performing operations. This can reduce latency and speed overall application performance for each application running on the cluster.
1. Namespace dns syntax record:
    1. Service - **service_name.namespace.cluster.local**
1. <https://www.youtube.com/watch?v=K3jNo4z5Jx8>
## RBAC
1. RBAC - <https://www.youtube.com/watch?v=Nw1ymxcLIDI>
1. Subject : Users, groups or service accounts.
1. Resources : Kubernetes API objects which we will operate on.
1. Verbs : The operations which we want to do with our resources.
1. Subject - Operation - Resource <https://kublr.com/wp-content/uploads/2020/06/Screen-Shot-2020-06-09-at-2.06.22-PM.png>
1. <http://rbac.dev/>
