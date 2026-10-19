# Kubernetes Setup on Arch Linux (Boot.dev Course)

The Boot.dev Kubernetes course uses **Minikube** with the **Docker** driver. This guide gets everything installed and running.

---

## 1. Install Docker, kubectl, and Minikube

```bash
sudo pacman -S docker kubectl minikube
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker
```

> Log out and back in if `newgrp docker` doesn't pick up the group change.

---

## 2. Verify Installation

```bash
docker run hello-world
kubectl version --client
minikube version
```

All three should print without errors.

---

## 3. Start Minikube

Use the exact command from the course (the CORS flag lets Boot.dev talk to your cluster):

```bash
minikube start --extra-config="apiserver.cors-allowed-origins=['http://boot.dev']"
```

- First run takes a few minutes (it downloads images).
- Success message: `Done! kubectl is now configured to use "minikube" cluster`.
- If the driver isn't auto-detected, add `--driver=docker`.

Check the cluster:

```bash
kubectl get nodes
```

You should see a `minikube` node with status `Ready`.

Now go back to Boot.dev and submit the CLI tests. **Keep minikube running for the rest of the course.**

---

## 4. Create the Course Repo

```bash
mkdir ~/k8s-course && cd ~/k8s-course
git init
```

Create a repo on GitHub, then:

```bash
git remote add origin git@github.com:<your-username>/k8s-course.git
git add .
git commit -m "initial commit"
git push -u origin main
```

Work from this directory for all course config files.

---

## 5. Handy Minikube Commands

| Command | What it does |
|---|---|
| `minikube status` | Check if the cluster is running |
| `minikube stop` | Pause the cluster (keeps state) |
| `minikube start` | Resume (flag not needed after first start) |
| `minikube delete` | Wipe the cluster and start fresh |
| `minikube dashboard` | Open the web UI |

---

## 6. Optional Quality-of-Life Setup

Add to `~/.bashrc` (or `~/.zshrc`, using `zsh` in place of `bash`):

```bash
source <(kubectl completion bash)
alias k=kubectl
complete -o default -F __start_kubectl k
```

Extra tools:

```bash
sudo pacman -S k9s helm
```

---

## 7. Essential kubectl Commands

```bash
kubectl get pods
kubectl describe pod <name>
kubectl logs <pod>
kubectl apply -f file.yaml
kubectl delete -f file.yaml
kubectl exec -it <pod> -- sh
```

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `permission denied` on docker socket | Re-login after `usermod -aG docker $USER` |
| `minikube start` fails | Run `systemctl status docker` and make sure Docker is running |
| Old/conflicting cluster | `minikube stop && minikube delete`, then start again |
| Config changes ignored | `minikube delete` and start again with the new flags |
| Leftover kind cluster confusing kubectl | `kind delete cluster --name learn` |
| Wrong kubectl context | `kubectl config use-context minikube` |

> Don't run minikube with `sudo`. Use the `docker` group instead.
