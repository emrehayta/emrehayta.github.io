---
title: "Essential Linux Commands: A Growing Cheat Sheet for Everyday Use"
date: 2024-09-28
tags: ["Linux", "Bash", "CLI", "Command Line", "Terminal", "Unix", "Debian"]
description: "A continually updated guide to essential Linux commands, covering file management, system monitoring, and networking with practical examples."
draft: false
---

## Introduction
Linux offers a multitude of powerful commands that make working on the command line efficient and effective. Whether you are managing files, monitoring system resources, or networking, knowing the right commands can save you a lot of time. This article introduces essential Linux commands and will be regularly updated with new commands and useful tips.

---

## File and Directory Management




### ls  
|              |                                             |
|--------------|---------------------------------------------|
| `ls -lah`    | List files and folder                       |
| `ls file*`   | List only entries beginning with `file`      |
| `ls *.txt`   | List only entries ending with `.txt`         |
| `-l`         | Long format: permissions, owner, group, size|
| `-a`         | Include all files: hidden files             |
| `-h`         | Human-readable: KB, MB, or GB format        |


### cd  
|              |                                             |
|--------------|---------------------------------------------|
| `cd /path/to/folder` | Change to the specified directory    |
| `cd ..`      | Go one directory up (parent directory)      |
| `cd -`       | Switch to the previous directory            |
| `cd ~`       | Switch to the home directory                |

### cp  
|              |                                             |
|--------------|---------------------------------------------|
| `cp file destination`      | Copy a file to the destination         |
| `cp -r folder destination` | Recursively copy a folder              |
| `cp -i source destination` | Prompt before overwriting              |
| `cp -u source destination` | Copy only if source is newer than destination |

### mv  
|              |                                             |
|--------------|---------------------------------------------|
| `mv source destination`    | Move or rename a file/folder          |
| `mv -i source destination` | Prompt before overwriting              |
| `mv -u source destination` | Move only if source is newer than destination |


---

## File and Text Manipulation
### cat  
|              |                                             |
|--------------|---------------------------------------------|
| `cat file`   | Display the contents of a file              |
| `cat file1 file2` | Concatenate and display multiple files |
| `cat > file` | Create a new file and write to it           |
| `cat >> file`| Append to an existing file                 |

### grep  
|              |                                             |
|--------------|---------------------------------------------|
| `grep 'pattern' file` | Search for a pattern in a file     |
| `grep -i 'pattern' file` | Case-insensitive search         |
| `grep -r 'pattern' folder` | Recursively search in a folder|
| `grep -v 'pattern' file` | Display lines not matching the pattern |

### head  
|              |                                             |
|--------------|---------------------------------------------|
| `head file`  | Display the first 10 lines of a file        |
| `head -n 20 file` | Display the first 20 lines of a file   |
| `head -c 50 file` | Display the first 50 bytes of a file   |
| `head -v file` | Always show file name before output       |

### tail  
|              |                                             |
|--------------|---------------------------------------------|
| `tail file`  | Display the last 10 lines of a file         |
| `tail -n 20 file` | Display the last 20 lines of a file    |
| `tail -f file` | Follow and display appended data in real-time |
| `tail -c 50 file` | Display the last 50 bytes of a file    |

---

## Advanced Text Manipulation

### awk  
|              |                                             |
|--------------|---------------------------------------------|
| `awk '{print $1}' file` | Print the first column of a file        |
| `awk -F, '{print $2}' file` | Use a comma as the field separator and print the second column |
| `awk '/pattern/' file` | Print lines that match a specific pattern |
| `awk 'NR==1 {print $0}' file` | Print only the first line of the file |

### sed  
|              |                                             |
|--------------|---------------------------------------------|
| `sed 's/old/new/g' file` | Replace all occurrences of "old" with "new" in a file |
| `sed -n '1,5p' file` | Print only lines 1 to 5 of a file        |
| `sed '/pattern/d' file` | Delete lines that match a specific pattern |
| `sed -i 's/old/new/g' file` | In-place replace "old" with "new" in the file |

---

## System Information and Process Management
### top  
|              |                                             |
|--------------|---------------------------------------------|
| `top`        | Display real-time system resource usage     |
| `top -u user`| Show processes for a specific user          |
| `top -p PID` | Monitor a specific process by PID           |
| `top -n 1`   | Display output once and exit                |

### htop  
|              |                                             |
|--------------|---------------------------------------------|
| `htop`       | Interactive process viewer with more details|
| `htop -u user` | Show processes for a specific user        |
| `htop -p PID`| Monitor a specific process by PID           |
| `htop --tree`| Display processes in a tree view            |

### df  
|              |                                             |
|--------------|---------------------------------------------|
| `df`         | Display disk space usage for all mounted filesystems |
| `df -h`      | Show disk space usage in human-readable format (e.g., MB, GB) |
| `df -T`      | Display filesystem type                     |
| `df -i`      | Show inode usage instead of block usage     |


---

## Networking
### ping  
|              |                                             |
|--------------|---------------------------------------------|
| `ping hostname_or_ip` | Send ICMP echo requests to test network connectivity |
| `ping -c 4 hostname_or_ip` | Send 4 ICMP echo requests and stop |
| `ping -i 2 hostname_or_ip` | Set the interval between sending packets to 2 seconds |
| `ping -t hostname_or_ip` | Ping continuously (until stopped manually) |

### ifconfig  
| Install: `sudo apt install net-tools`        |
|--------------|---------------------------------------------|
| `ifconfig`   | Display network interfaces and their configurations |
| `ifconfig eth0` | Show details for a specific interface (e.g., eth0) |
| `ifconfig eth0 up` | Enable the specified interface         |
| `ifconfig eth0 down` | Disable the specified interface      |

### curl  
| Install: `sudo apt install curl`             | |
|--------------|---------------------------------------------|
| `curl url`   | Fetch the content of the specified URL      |
| `curl -O url`| Download a file from the URL                |
| `curl -I url`| Fetch only the headers of the specified URL |
| `curl -d "data" url` | Send POST data to the specified URL |

### netstat  
|              |                                             |
|--------------|---------------------------------------------|
| `netstat`    | Display network connections, routing tables, interface statistics |
| `netstat -tuln` | Show listening TCP and UDP ports         |
| `netstat -i` | Display network interface statistics        |
| `netstat -r` | Display the kernel routing table            |

### nc (netcat)  
|              |                                             |
|--------------|---------------------------------------------|
| `nc -l 12345` | Listen on port 12345 for incoming connections |
| `nc -v hostname_or_ip 12345` | Connect to a specified host and port |
| `nc -zv hostname_or_ip 80` | Check if a specific port is open (e.g., port 80) |
| `nc -w 5 hostname_or_ip 12345` | Set a timeout of 5 seconds for the connection |

### telnet  
| Install: `sudo apt install telnet`|   |
|--------------|---------------------------------------------|
| `telnet hostname_or_ip port` | Open a connection to a specific host and port |
| `telnet localhost 25` | Test connection to a local mail server (port 25) |
| `telnet hostname_or_ip` | Open a connection to a specific host (default port 23) |
| `telnet ?`  | Display available telnet commands            |

### dig  
| Install: `sudo apt install dnsutils`|   |
|--------------|---------------------------------------------|
| `dig domain` | Perform a DNS lookup for the specified domain |
| `dig +short domain` | Display a simplified output (just the IP) |
| `dig @dns_server domain` | Perform a DNS lookup using a specific DNS server |
| `dig -x ip_address` | Perform a reverse DNS lookup for the IP address |

### nslookup  
|              |                                             |
|--------------|---------------------------------------------|
| `nslookup domain` | Query DNS information for the specified domain |
| `nslookup` | Enter interactive mode to perform multiple queries |
| `nslookup domain dns_server` | Query the domain using a specific DNS server |
| `nslookup -type=any domain` | Query for all available DNS record types |



---



## Updates
- **September 27, 2024**: Added section on networking commands (`ping`, `curl`).
- **October 15, 2024**: Added new examples for `grep` and `awk`.
- **October 18, 2024**: Updated structure of blog article
