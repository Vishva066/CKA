# POD Security

Pod Security is a built-in Kubernetes mechanism (as of v1.25+) to enforce security standards for Pods at the namespace level. It controls what a Pod is allowed to do based on how it's configured — for example, whether it runs as root, uses host networking, mounts host paths, etc.


Pod Security Standards (PSS) give you three fairly broad policy enforcement capabilities.

1. Priviledged -> Unrestricted policy, providing the widest possible level of permissions. This policy allows for known privilege escalations.

2. Baseline -> Minimally restrictive policy which prevents known privilege escalations. Allows the default (minimally specified) Pod configuration.

3. Restricted -> Heavily restricted policy, following current Pod hardening best practices.



