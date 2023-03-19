Title:
Node metrics logs should not be public

Impact:
From Metrics logs, the attacker can obtain system variables, path parameters, etc.

Solution:
It was a good practice that do not exposed metrics in a non-production environment. 

Poc:
```
http://34.230.175.149:4000/metrics
http://34.230.175.149/metrics
https://evm.explorer.canto.io/metrics
https://epotato.canto.io/metrics
https://epotato.explorer.canto.io/metrics
https://eevm.explorer.canto.io/metrics
```
```
erlang_vm_allocators{alloc="driver_alloc",instance_no="13",kind="mbcs_pool",usage="blocks_size"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="13",kind="sbcs",usage="carriers"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="13",kind="sbcs",usage="carriers_size"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="13",kind="sbcs",usage="blocks"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="13",kind="sbcs",usage="blocks_size"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="14",kind="mbcs",usage="carriers"} 1
erlang_vm_allocators{alloc="driver_alloc",instance_no="14",kind="mbcs",usage="carriers_size"} 32768
erlang_vm_allocators{alloc="driver_alloc",instance_no="14",kind="mbcs",usage="blocks"} 7
erlang_vm_allocators{alloc="driver_alloc",instance_no="14",kind="mbcs",usage="blocks_size"} 3472
erlang_vm_allocators{alloc="driver_alloc",instance_no="14",kind="mbcs_pool",usage="carriers"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="14",kind="mbcs_pool",usage="carriers_size"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="14",kind="mbcs_pool",usage="blocks"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="14",kind="mbcs_pool",usage="blocks_size"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="14",kind="sbcs",usage="carriers"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="14",kind="sbcs",usage="carriers_size"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="14",kind="sbcs",usage="blocks"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="14",kind="sbcs",usage="blocks_size"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="15",kind="mbcs",usage="carriers"} 1
erlang_vm_allocators{alloc="driver_alloc",instance_no="15",kind="mbcs",usage="carriers_size"} 32768
erlang_vm_allocators{alloc="driver_alloc",instance_no="15",kind="mbcs",usage="blocks"} 7
erlang_vm_allocators{alloc="driver_alloc",instance_no="15",kind="mbcs",usage="blocks_size"} 3472
erlang_vm_allocators{alloc="driver_alloc",instance_no="15",kind="mbcs_pool",usage="carriers"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="15",kind="mbcs_pool",usage="carriers_size"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="15",kind="mbcs_pool",usage="blocks"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="15",kind="mbcs_pool",usage="blocks_size"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="15",kind="sbcs",usage="carriers"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="15",kind="sbcs",usage="carriers_size"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="15",kind="sbcs",usage="blocks"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="15",kind="sbcs",usage="blocks_size"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="16",kind="mbcs",usage="carriers"} 1
erlang_vm_allocators{alloc="driver_alloc",instance_no="16",kind="mbcs",usage="carriers_size"} 32768
erlang_vm_allocators{alloc="driver_alloc",instance_no="16",kind="mbcs",usage="blocks"} 9
erlang_vm_allocators{alloc="driver_alloc",instance_no="16",kind="mbcs",usage="blocks_size"} 4960
erlang_vm_allocators{alloc="driver_alloc",instance_no="16",kind="mbcs_pool",usage="carriers"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="16",kind="mbcs_pool",usage="carriers_size"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="16",kind="mbcs_pool",usage="blocks"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="16",kind="mbcs_pool",usage="blocks_size"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="16",kind="sbcs",usage="carriers"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="16",kind="sbcs",usage="carriers_size"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="16",kind="sbcs",usage="blocks"} 0
erlang_vm_allocators{alloc="driver_alloc",instance_no="16",kind="sbcs",usage="blocks_size"} 0

```