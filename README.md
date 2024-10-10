If you are wondering which process is causing your gpu to not go into S0ix state or suspended, this is for you. 

It tracks down the PIDs of the PIDs using your NVIDIA card.

Great for squeezing out more battery life from a laptop who refuses to suspend the NVIDIA card.


```bash
#!/bin/bash
#auth: Deniz Canbay
echo "=== NVIDIA GPU STATS ==="
gpu_dirs=(/proc/driver/nvidia/gpus/*)

if [ -e "${gpu_dirs[0]}" ]; then
    gpu_num=1
    for gpu_dir in "${gpu_dirs[@]}"; do
        busid=$(basename "$gpu_dir")
        busid=${busid#PCI:}

        echo "NVIDIA CARD $gpu_num"
        echo "Card Bus ID: $busid"
        sysfs_path="/sys/bus/pci/devices/$busid"
        if [ -f "$sysfs_path/power/runtime_status" ]; then
            echo "Executing command: cat $sysfs_path/power/runtime_status"
            cat "$sysfs_path/power/runtime_status"
        else
            echo "Runtime status not found for device $busid"
        fi
        echo
        echo "Executing command: cat $gpu_dir/power"
        if [ -f "$gpu_dir/power" ]; then
            cat "$gpu_dir/power"
        else
            echo "Power status not found for device $busid"
        fi

        echo

        gpu_num=$((gpu_num +1))
    done
else
    echo "No NVIDIA GPUs detected."
fi

echo "Processes using NVIDIA:"

if compgen -G "/dev/nvidia*" > /dev/null; then
    PIDS=$(lsof -t /dev/nvidia* 2>/dev/null | sort -u)
else
    PIDS=""
fi

if [ -z "$PIDS" ]; then
    echo "No processes are using /dev/nvidia* devices."
else
    for PID in $PIDS; do
        echo
        echo "Process using NVIDIA device:"
        ps -fp "$PID"

        if [ -n "$PPID" ]; then
            echo "Parent process:"
            ps -fp "$PPID"
        fi
    done
fi
```
