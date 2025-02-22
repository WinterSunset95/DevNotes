# Linux related knowledge - Commands, Tips and Tricks
- [Change ownership of file](#change-ownership-of-file)
- [Find running services](#find-running-services)
- [Kill processes based on PID](#kill-processes-based-on-pid)

## Change ownership of file
```bash
# Change ownership of one file
sudo chown user:group filename

# Recursively
sudo chown -hR user:group foldername/
```

---
## Find running services
```bash
sudo ps aux | grep *name*
```
For example, to find all running instances of *dnsmasq*
```bash
sudo ps aux | grep dnsmasq
```

---
## Kill processes based on PID
```bash
sudo kill pid
```

---
## Make temp dir
```bash
mktemp -d -t XXXXXXXXXXXX
```

---
## Check if a folder exists in bash
```bash
mkdir -p dir
```

---
## Check if a file exists in bash
```bash
if [ -f /path/to/file ]; then
    echo "File exists"
fi
```

---
## Declare a function in bash
