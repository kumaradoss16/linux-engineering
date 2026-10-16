## Practical Linux lab (cut)

hytrel@Ubuntu-Server:~/fs-lab$ mkdir -p ~/fs-lab/cut-practice
hytrel@Ubuntu-Server:~/fs-lab$ cd ~/fs-lab/cut-practice
hytrel@Ubuntu-Server:~/fs-lab/cut-practice$ printf 'hytrel:1000:Ubuntu\nalice:1001:Debain\nbob:1002:Fedore\n' >users.txt

# Display the file
hytrel@Ubuntu-Server:~/fs-lab/cut-practice$ cat users.txt
hytrel:1000:Ubuntu
alice:1001:Debain
bob:1002:Fedore

# Extract usernames
hytrel@Ubuntu-Server:~/fs-lab/cut-practice$ cut -d ':' -f 1 users.txt
hytrel
alice
bob

# Extract usernames and UIDs
hytrel@Ubuntu-Server:~/fs-lab/cut-practice$ cut -d ':' -f 1,2 users.txt
hytrel:1000
alice:1001
bob:1002

# Extract the operating system field
hytrel@Ubuntu-Server:~/fs-lab/cut-practice$ cut -d ':' -f 3 users.txt
Ubuntu
Debain
Fedore

# Extract all fields except the first
hytrel@Ubuntu-Server:~/fs-lab/cut-practice$ cut -d ':' --complement -f 1 users.txt
1000:Ubuntu
1001:Debain
1002:Fedore
