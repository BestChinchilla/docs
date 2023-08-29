```bash
yc compute instance list
```

Result:

```text
+----------------------+---------+---------------+---------+--------------+-------------+
|          ID          |  NAME   |    ZONE ID    | STATUS  | EXTERNAL IP  | INTERNAL IP |
+----------------------+---------+---------------+---------+--------------+-------------+
| jklp0o9i8012******** | my-vm-1 | {{ region-id }}-b | RUNNING | 51.250.**.** | 192.168.*.* |
| mnoa5s6d8345******** | my-vm-2 | {{ region-id }}-b | RUNNING | 84.201.**.** | 192.168.*.* |
+----------------------+---------+---------------+---------+--------------+-------------+
```