---
title: Shell Scripts
draft: false
tags:
  - automation
	- scripts
	- shell
---


Using the shell is the easiest and most direct means of accomplishing anything on a computational system. It is the primary means by which humans interface with the system. 

Good scripts are well documented, easy to read, aim for simplicity, and avoids errors. 

While there are many different shells out there, Bash is the most common. So, write your shell scripts in bash.

> [!INFO]
> The native shell of OpenBSD is KSH, while FreeBSD uses SH version: 1998, MacOS now use ZSH, and Alpine Linux uses ASH.

A good template to develop programs written in shell is [Bashew](https://github.com/pforret/bashew). 

```bash
#!/usr/bin/env bash
# vim: set sw=4 sts=4 et ft=sh :
# -*- mode:bash; -*-
# -----------------------------------------------
VERSION="0.3.0"
LOGDIR="$HOME/log"
REM_USER="USER"
LOCPORT="5900"
REMPORT="5900"
VNC_USERNAME="USER"
VNC_PASSWORD="PASSWORD"
TUN="${LOCPORT}:localhost:${REMPORT}"
TARGET="$REM_USER@192.168.XXX.XXX"
PWFILE="$XDG_CONFIG/tigervnc/passwd"

SSH_PIDFILE="$HOME/.cache/tigervnc-ssh.pid"
VNC_PIDFILE="$HOME/.cache/tigervnc-viewer.pid"

export VNC_USERNAME
export VNC_PASSWORD

# ---------- Required depends ----------
[[-f $(which "dunstify")]] || echo "Dunstify not installed" && exit 1

# ---------- Log ----------
setup_log () {
	source "$HOME/bin/logging.sh"
	init_logger -d 7 -l "$LOGDIR/ltx-motions.log"
	log_info "Script Started"
}

# ---------- helpers ----------
is_running() {
	local pid=$1
	[[ -n $pid ]] && kill -0 "$pid" 2>/dev/null
}

save_pid() {
	local pid=$1 file=$2
	mkdir -p "$(dirname "$file")"
	echo "$pid" > "$file"
}

read_pid() {
	local file=$1
	[[ -f $file ]] && cat "$file" || echo ""
}

# ---------- Create Tunnel ----------
create_tunnel() {
	log_debug "Creating Tunnel $TUN -> $TARGET"
	ssh -N -T -L "$TUN" "$TARGET" &>/dev/null &
	SSHPID=$!
	save_pid "$SSHPID" "$SSH_PIDFILE"
	# Give the tunnel a moment to come up
	sleep 1
	if is_running "$SSHPID"; then
		log_notice "SSH tunnel started (PID $SSHPID)"
		dunstify "Tunnel Started" "SSH PID $SSHPID"
	else
		log_error "Failed to start SSH tunnel"
		dunstify "Tunnel Failed" "Could not start SSH tunnel"
		return 1
	fi
}

# ---------- viewer ----------
start_viewer() {
	# Connect to the *local* side of the tunnel
	vncviewer -SecurityTypes VncAuth \
	          -ReconnectOnError \
	          -passwd "$PWFILE" \
	          localhost:"$LOCPORT" &>/dev/null &
	VNCPID=$!
	save_pid "$VNCPID" "$VNC_PIDFILE"

	log_notice "vncviewer started (PID $VNCPID)"
	dunstify "VNC Started" "Viewer PID $VNCPID"
}

# ---------- monitoring ----------
watch_vnc() {
	while is_running "$VNCPID"; do
		sleep 5
	done
	log_info "vncviewer exited — cleaning up"
	notify_kill
}

# ---------- cleanup ----------
notify_kill() {
	log_info "Cleaning up tunnel and remote sessions"

	# Kill the SSH tunnel we started
	if is_running "$SSHPID"; then
		kill "$SSHPID" 2>/dev/null
		wait "$SSHPID" 2>/dev/null
		log_info "SSH tunnel $SSHPID killed"
	fi
	rm -f "$SSH_PIDFILE"

	# Kill the viewer if it is still around
	if is_running "$VNCPID"; then
		kill "$VNCPID" 2>/dev/null
		wait "$VNCPID" 2>/dev/null
	fi
	rm -f "$VNC_PIDFILE"

	dunstify "VNC Stopped" "Tunnel and viewer cleaned up"
}

# ---------- Killing ----------
kill_vnc() {
	# Prefer stored PIDs; fall back to pgrep only if necessary
	SSHPID=$(read_pid "$SSH_PIDFILE")
	VNCPID=$(read_pid "$VNC_PIDFILE")

	if ! is_running "$SSHPID" && ! is_running "$VNCPID"; then
		dunstify "Nothing Running" "No VNC tunnel or viewer found"
		exit 0
	fi

	notify_kill
}

# ---------- Status ----------
vnc_status() {
	SSHPID=$(read_pid "$SSH_PIDFILE")
	VNCPID=$(read_pid "$VNC_PIDFILE")

	if is_running "$SSHPID"; then
		dunstify "SSH Tunnel" "Running (PID $SSHPID)"
	else
		dunstify "SSH Tunnel" "Not running"
	fi

	if is_running "$VNCPID"; then
		dunstify "VNC Viewer" "Running (PID $VNCPID)"
	else
		dunstify "VNC Viewer" "Not running"
	fi
}

# ---------- main entry ----------
start-vnc() {
	# Load any previously recorded PIDs
	SSHPID=$(read_pid "$SSH_PIDFILE")
	VNCPID=$(read_pid "$VNC_PIDFILE")

	if ! is_running "$SSHPID"; then
		create_tunnel || return 1
	else
		log_info "Reusing existing tunnel (PID $SSHPID)"
	fi

	sleep 1
	start_viewer
	watch_vnc
}

# ---------- Print Help ----------
print_help() {
	cat <<- HELP
Usage: $0 { start | stop | status | version | help }

start   - Start SSH tunnel + vncviewer
stop    - Kill tunnel and viewer
status  - Show current PIDs
version - Print version
help    - This help
HELP
}

# ---------- Switch ----------
main() {
	case $1 in
		start)
			setup_log
			start-vnc
			;;
		stop)
			setup_log
			kill_vnc
			;;
		status)
			vnc_status
			;;
		version)
			echo "$VERSION"
			;;
		*)
			print_help
			;;
	esac
}

main "$@"

```
