
# Exercise 18-2: Generate, Distribute, and Use SSH Keys
1. Generate RSA keys without a password  and without detailed output
```bash
ssh-keygen -N "" -q
```

2. Show public and private keys with cat
```bash
cat /.ssh/id_rsa
cat /.ssh/id_rsa.pub
```
3. Copy the public key file to server20 
```bash
ssh-copy-id server20
```
4. On serv10, run ssh command as user1 to connect to server20
```bash
ssh server20
```
5. ljds;lfjsdjfjSLFDJLSDjfl