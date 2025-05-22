# Pod Security Admission

Pod Security admission places requirements on a Pod's Security Context and other related fields according to the three levels defined by the Pod Security Standards: privileged, baseline, and restricted.

Kubernetes defines a set of labels that you can set to define which of the predefined Pod Security Standard levels you want to use for a namespace. The label you select defines what action the control plane takes if a potential violation is detected:


- enforce -> Policy violations will cause the pod to be rejected.

- audit -> Policy violations will trigger the addition of an audit annotation to the event recorded in the audit log, but are otherwise allowed.

warn -> Policy violations will trigger a user-facing warning, but are otherwise allowed.

