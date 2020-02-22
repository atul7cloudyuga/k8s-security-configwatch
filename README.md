# Kubernetes Security Lint

This Git Action run security lint check against Kubernetes workloads in Git workflow (PR open, commit pushed etc.).

## Inputs

### `sourceDir`

**Required** The source directory for k8s workload yaml. (master branch)

### `targetDir`

**Required** The target directory for k8s workload yaml. (PR branch)

## Use Cases
0. Integrate the `k8s-security-lint` action into the git workflow.
1. Examine the following security attributes changes in k8s workload YAMLs in a PR:
- `Privileged`
- `HostPID`
- `HostIPC`
- `HostNetwork`
- `Capabilities`
- `ReadOnlyRootFileSystem`
- `RunAsUser` (root/nonroot)
- `RunAsGroup` (root/nonroot)
- volume types
2. Define your own criteria based on the lint result, for example:
- Send lint report to slack channel if `privileged` mode is set to true
- Fail the check on the PR if some host level namespaces are enabled. (`hostNetwork` etc.)
- Assign extra reviewers (security architect/engineer) to the PR.

## Example Usage in Git workflow
```
# checkout master branch
- uses: actions/checkout@v2
    with:
      ref: master
      path: master
# checkout PR branch
- uses: actions/checkout@v2
    with:
      path: candidate
      ref: ${{ github.event.pull_request.head.sha }}
# pass the yamls directory to k8s-privilege-check git action
- name: Kubernetes Security Lint
  uses: sysdiglabs/k8s-security-lint@v1.0.0
  with:
    sourceDir: '/master/yamls'
    targetDir: '/candidate/yamls'
# evaluate escalation report
- name: Post Privilege Check
  run: |
    echo ${{ toJSON(steps.k8s_privilege_check.outputs.escalation_report) }}
    # slack
    # or other git action like adding another reviewer
```


## Outputs

### `escalation_report`
```
{
  "total_source_workloads": 2,
  "total_target_workloads": 2,
  "total_source_images": 2,
  "total_target_images": 2,
  "escalation_count": 2,
  "reduction_count": 1,
  "escalations": [
    {
      "name": "nginx",
      "kind": "Pod",
      "namespace": "default",
      "file": "nginx.yaml"
    },
    {
      "name": "my-busybox",
      "kind": "Pod",
      "namespace": "psp-test",
      "file": "busy-box.yaml"
    }
  ],
  "reductions": [
    {
      "name": "my-busybox",
      "kind": "Pod",
      "namespace": "psp-test",
      "file": "busy-box.yaml"
    }
  ],
  "new_privileged": {
    "status": "Escalated",
    "previous": "false",
    "current": "true",
    "workloads": [
      {
        "name": "nginx",
        "kind": "Pod",
        "namespace": "default",
        "file": "nginx.yaml",
        "image": "kaizheh/nginx"
      }
    ],
    "workloads_count": 1
  },
  "removed_privileged": {
    "status": "Reduced",
    "previous": "true",
    "current": "false",
    "workloads": [
      {
        "name": "my-busybox",
        "kind": "Pod",
        "namespace": "psp-test",
        "file": "busy-box.yaml",
        "image": "busybox"
      }
    ],
    "workloads_count": 1
  },
  "new_hostIPC": {
    "status": "Escalated",
    "previous": "false",
    "current": "true",
    "workloads": [
      {
        "name": "my-busybox",
        "kind": "Pod",
        "namespace": "psp-test",
        "file": "busy-box.yaml"
      },
      {
        "name": "nginx",
        "kind": "Pod",
        "namespace": "default",
        "file": "nginx.yaml"
      }
    ],
    "workloads_count": 2
  },
  "removed_hostIPC": {
    "status": "Reduced",
    "previous": "true",
    "current": "false",
    "workloads": [],
    "workloads_count": 0
  },
  "new_hostNetwork": {
    "status": "Escalated",
    "previous": "false",
    "current": "true",
    "workloads": [
      {
        "name": "nginx",
        "kind": "Pod",
        "namespace": "default",
        "file": "nginx.yaml"
      }
    ],
    "workloads_count": 1
  },
  "removed_hostNetwork": {
    "status": "Reduced",
    "previous": "true",
    "current": "false",
    "workloads": [
      {
        "name": "my-busybox",
        "kind": "Pod",
        "namespace": "psp-test",
        "file": "busy-box.yaml"
      }
    ],
    "workloads_count": 1
  },
  "new_hostPID": {
    "status": "Escalated",
    "previous": "false",
    "current": "true",
    "workloads": [
      {
        "name": "nginx",
        "kind": "Pod",
        "namespace": "default",
        "file": "nginx.yaml"
      }
    ],
    "workloads_count": 1
  },
  "removed_hostPID": {
    "status": "Reduced",
    "previous": "true",
    "current": "false",
    "workloads": [
      {
        "name": "my-busybox",
        "kind": "Pod",
        "namespace": "psp-test",
        "file": "busy-box.yaml"
      }
    ],
    "workloads_count": 1
  },
  "new_volume_types": {
    "hostPath": {
      "status": "Escalated",
      "previous": "",
      "current": "hostPath",
      "workloads": [
        {
          "name": "nginx",
          "kind": "Pod",
          "namespace": "default",
          "file": "nginx.yaml"
        }
      ],
      "workloads_count": 1
    }
  },
  "removed_volume_types": {},
  "new_capabilities": {},
  "reduced_capabilities": {
    "SYS_ADMIN": {
      "status": "Reduced",
      "previous": "SYS_ADMIN",
      "current": "",
      "workloads": [
        {
          "name": "my-busybox",
          "kind": "Pod",
          "namespace": "psp-test",
          "file": "busy-box.yaml",
          "image": "busybox"
        }
      ],
      "workloads_count": 1
    },
    "SYS_CHROOT": {
      "status": "Reduced",
      "previous": "SYS_CHROOT",
      "current": "",
      "workloads": [
        {
          "name": "my-busybox",
          "kind": "Pod",
          "namespace": "psp-test",
          "file": "busy-box.yaml",
          "image": "busybox"
        }
      ],
      "workloads_count": 1
    }
  },
  "new_run_user_as_root": {
    "status": "Escalated",
    "previous": "non-root",
    "current": "root",
    "workloads": [
      {
        "name": "nginx",
        "kind": "Pod",
        "namespace": "default",
        "file": "nginx.yaml",
        "image": "kaizheh/nginx"
      }
    ],
    "workloads_count": 1
  },
  "removed_run_user_as_root": {
    "status": "Reduced",
    "previous": "root",
    "current": "non-root",
    "workloads": [],
    "workloads_count": 0
  },
  "new_run_group_as_root": {
    "status": "Escalated",
    "previous": "non-root",
    "current": "root",
    "workloads": [
      {
        "name": "nginx",
        "kind": "Pod",
        "namespace": "default",
        "file": "nginx.yaml",
        "image": "kaizheh/nginx"
      }
    ],
    "workloads_count": 1
  },
  "removed_run_group_as_root": {
    "status": "Reduced",
    "previous": "root",
    "current": "non-root",
    "workloads": [],
    "workloads_count": 0
  },
  "new_read_only_root_fs": {
    "status": "Reduced",
    "previous": "false",
    "current": "true",
    "workloads": [
      {
        "name": "my-busybox",
        "kind": "Pod",
        "namespace": "psp-test",
        "file": "busy-box.yaml",
        "image": "busybox"
      }
    ],
    "workloads_count": 1
  },
  "removed_read_only_root_fs": {
    "status": "Escalated",
    "previous": "true",
    "current": "false",
    "workloads": [
      {
        "name": "nginx",
        "kind": "Pod",
        "namespace": "default",
        "file": "nginx.yaml",
        "image": "kaizheh/nginx"
      }
    ],
    "workloads_count": 1
  }
}
```
The above escalation report is generated in [PR](https://github.com/Kaizhe/k8s-workloads/pull/13)
