# Docker and Splunk Command Sheet

Quick reference for running Splunk Enterprise in a Docker container on Apple Silicon (M2 MacBook Air).

---

## 1. Creating the container (first time only)

Run this once to create the container. After this, use `start` and `stop` instead.

```bash
docker run -d --platform linux/amd64 \
  --name splunk \
  -p 8000:8000 \
  -p 8088:8088 \
  -p 8089:8089 \
  -e SPLUNK_GENERAL_TERMS='--accept-sgt-current-at-splunk-com' \
  -e SPLUNK_START_ARGS='--accept-license' \
  -e SPLUNK_PASSWORD='<your-password>' \
  splunk/splunk:latest
```

### What each flag does

| Flag | Meaning |
|---|---|
| `-d` | Detached. Runs in the background so your terminal stays free. |
| `--platform linux/amd64` | Runs the Intel image under emulation. Splunk has no ARM64 build. |
| `--name splunk` | Names the container so you can refer to it by name, not ID. |
| `-p 8000:8000` | Web interface. Maps Mac port 8000 to container port 8000. |
| `-p 8088:8088` | HTTP Event Collector (HEC). |
| `-p 8089:8089` | Splunk management port, used by the CLI and REST API. |
| `-e SPLUNK_GENERAL_TERMS` | Accepts the Splunk General Terms. Required on newer images. |
| `-e SPLUNK_START_ARGS` | Accepts the licence. |
| `-e SPLUNK_PASSWORD` | Sets the admin password at first boot. |
| `splunk/splunk:latest` | The image to run. Must be the last argument. |

**Password rules:** minimum 8 characters with upper, lower, digit, and special character. Splunk will refuse to start if the password fails the policy.

**Never commit the real password to GitHub.** Replace it with `<your-password>` in any documentation.

---

## 2. Daily use

```bash
# Start when resuming work
docker start splunk

# Stop when finished for the day
docker stop splunk

# Restart (stop plus start in one)
docker restart splunk
```

Wait one to three minutes after starting before opening the browser. Emulation makes boot slower.

Web interface: `http://localhost:8000`
Login: `admin` plus the password you set.

---

## 3. Checking status

```bash
# List running containers
docker ps

# List all containers including stopped ones
docker ps -a

# Watch startup logs live (Ctrl+C to stop watching)
docker logs -f splunk

# Last 50 log lines
docker logs --tail 50 splunk
```

Look for `(healthy)` in the STATUS column of `docker ps`. That means the web interface is actually responding, not just that the container is on.

---

## 4. Running Splunk CLI commands inside the container

**Always include `--user splunk`.** Without it, Docker runs as root, which cannot access Splunk's own files, and you get a wall of "Permission denied" errors.

```bash
docker exec -it --user splunk splunk /opt/splunk/bin/splunk <command>
```

The first `splunk` after `--user` is the username. The second is the container name.

### Common ones

```bash
# Check whether splunkd is running
docker exec -it --user splunk splunk /opt/splunk/bin/splunk status

# Stop Splunk (not the container)
docker exec -it --user splunk splunk /opt/splunk/bin/splunk stop

# Start Splunk
docker exec -it --user splunk splunk /opt/splunk/bin/splunk start

# Open a shell inside the container
docker exec -it --user splunk splunk bash
```

---

## 5. Wiping indexed data

Splunk must be stopped before cleaning. This is destructive and cannot be undone.

```bash
# Wipe everything
docker exec -it --user splunk splunk /opt/splunk/bin/splunk stop
docker exec -it --user splunk splunk /opt/splunk/bin/splunk clean eventdata -f
docker exec -it --user splunk splunk /opt/splunk/bin/splunk start

# Wipe a single index
docker exec -it --user splunk splunk /opt/splunk/bin/splunk clean eventdata -index web -f
```

Index definitions survive a clean. Only the data inside them is removed, so you do not need to recreate the index afterwards.

---

## 6. Uploading data via CLI (bypasses the preview wizard)

Useful when the data preview screen fails or when you want to script an upload.

```bash
docker exec -it --user splunk splunk /opt/splunk/bin/splunk add oneshot /path/inside/container/secure.log \
  -index security \
  -sourcetype linux_secure \
  -host web1 \
  -auth admin:<your-password>
```

The path must be a path **inside the container**, not on your Mac. This only works if the file is reachable from inside, which needs a volume mount.

---

## 7. Volume mounts (optional)

Volumes connect a folder on your Mac to a folder inside the container.

```bash
# Make practice data visible inside the container
-v /Users/<you>/Desktop/Practice_Data:/practice_data

# Keep indexed data alive even if the container is deleted
-v splunk-data:/opt/splunk/var
```

**Trade-off:** the second one makes first boot significantly slower, because Docker copies Splunk's existing `var` directory into the new volume before starting. This happens once only.

**Not needed for the course.** The upload wizard's "upload from your computer" option uses your browser's file picker, which already reads your Mac's filesystem directly.

---

## 8. Port conflicts

If port 8000 is already taken:

```bash
# Find what is using it
lsof -nP -iTCP:8000 -sTCP:LISTEN

# Check for a container holding it
docker ps -a --filter "publish=8000"
```

Either free the port, or map a different one when creating the container:

```bash
-p 8001:8000
```

Reads as: host port 8001 maps to container port 8000. Splunk still runs on 8000 internally, and you reach it at `http://localhost:8001`.

---

## 9. Cleanup

```bash
# Delete the container (DESTROYS ALL INDEXED DATA unless you used a named volume)
docker rm splunk

# Force delete a running container
docker rm -f splunk

# Remove the Splunk image to free disk space
docker rmi splunk/splunk:latest

# List named volumes
docker volume ls
```

**The rule:** `stop` and `start` are safe. `rm` is not.

---

## 10. If you also have a native Splunk install

Both want port 8000, so only one can run at a time.

```bash
# Stop the native install
/Applications/Splunk/bin/splunk stop

# Prevent it launching at boot
sudo /Applications/Splunk/bin/splunk disable boot-start
```

---

## Quick troubleshooting table

| Symptom | Likely cause | Fix |
|---|---|---|
| Permission denied errors from `docker exec` | Running as root | Add `--user splunk` |
| "Port already bound" | Native Splunk or old container running | `lsof -nP -iTCP:8000 -sTCP:LISTEN` |
| "License not accepted" | Missing `SPLUNK_GENERAL_TERMS` | Add both licence env vars |
| Container name already in use | Failed container still exists | `docker rm splunk` first |
| Very slow first boot | Named volume copying `var` | Expected once. Wait it out or drop the volume. |
| Browser shows connection error | Still booting | Wait for `(healthy)` in `docker ps` |
| Searches return nothing | Time range set to Last 24 hours | Change time picker to All time |
