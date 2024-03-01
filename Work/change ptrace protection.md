cat /proc/sys/kernel/yama/ptrace_scope

temp change:
echo 0 | sudo tee /proc/sys/kernel/yama/ptrace_scope

permanent change:
echo "kernel.yama.ptrace_scope = 0" | sudo tee -a /etc/sysctl.d/10-ptrace.conf
sudo sysctl -p /etc/sysctl.d/10-ptrace.conf
