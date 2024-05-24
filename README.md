# zsh
```
# Set Zsh as default shell
export SHELL=$(which zsh)
export TERM="xterm-256color"

# Path to your oh-my-zsh installation.
export ZSH="$HOME/.oh-my-zsh"

# Enable the necessary plugins
plugins=(
    git
    virtualenv
    battery
    docker
    node
    aliases
    copypath
    history
    github
    composer
    laravel
    brew
    zsh-completions
    zsh-autosuggestions
    zsh-syntax-highlighting
)

source $ZSH/oh-my-zsh.sh

# Function to get virtualenv info
virtualenv_info() {
    if [[ -n "$VIRTUAL_ENV" ]]; then
        echo "(virtualenv $(basename $VIRTUAL_ENV))"
    fi
}

# Function to get conda environment info
conda_env_info() {
    if command -v conda &> /dev/null; then
        local conda_env=$(conda info --envs | grep '*' | awk '{print $1}')
        if [[ -n "$conda_env" ]]; then
            echo "(conda $conda_env)"
        fi
    fi
}

# Function to get battery level (Linux)
battery_info() {
    if command -v upower &> /dev/null; then
        local battery_level=$(upower -i $(upower -e | grep battery) | grep -E "percentage" | awk '{print $2}')
        if [[ -n "$battery_level" ]]; then
            echo "[Battery: $battery_level]"
        else
            echo "[Battery: ?%]"
        fi
    else
        echo "[Battery: ?%]"
    fi
}

# Custom prompt setup
precmd() {
    # Getting current date and time in Tokyo and UTC
    local datetime_tokyo=$(TZ=Asia/Tokyo date +"%Y/%m/%d %H:%M:%S")
    local datetime_utc=$(date -u +"%Y/%m/%d %H:%M:%S")

    # Getting Git branch info
    local git_branch=$(git rev-parse --abbrev-ref HEAD 2>/dev/null)
    local git_changes=$(git status --porcelain 2>/dev/null | wc -l)

    # Getting Python version
    local python_version=$(python --version 2>&1)

    # Getting Node.js version
    local node_version=$(node -v 2>/dev/null)

    # Getting React version
    local react_version=$(npm list react | grep react | awk -F@ '{print $2}' 2>/dev/null)

    # Getting Docker containers status
    local docker_status=$(docker ps --format "{{.Names}}: {{.Status}}" 2>/dev/null | paste -sd " " -)

    # Getting local IP address
    local local_ip=$(hostname -I | awk '{print $1}')

    # Getting memory usage
    local mem_usage=$(free -m | awk '/^Mem:/{printf "%s/%sMB (%.2f%%)", $3, $2, $3/$2 * 100.0}')

    # Getting disk usage
    local disk_usage=$(df -h / | awk 'NR==2 {printf "%s/%s (%s)", $3, $2, $5}')

    # Getting CPU usage
    local cpu_usage=$(top -bn1 | grep "Cpu(s)" | sed "s/.*, *\([0-9.]*\)%* id.*/\1/" | awk '{printf "%.2f", 100 - $1}')

    # Setting the prompt
    PROMPT="%F{blue}┌─[%n@%m]─[$datetime_tokyo (東京時間)] [$datetime_utc (UTC)]%f%F{red}%(?..[ERROR])%f
%F{yellow}├─[%~]%f
%F{green}├─[on ${git_branch:-not a git repo}-(${git_changes} changes)]-$(virtualenv_info)-$(conda_env_info)-[Python ${python_version}]-[Node ${node_version}]-[React ${react_version}]-[Docker ${docker_status}]%f
%F{cyan}├─[IP : ${local_ip}]-[Mem : ${mem_usage}]-[Disk : ${disk_usage}]-[CPU : ${cpu_usage} ]-$(battery_info)
└──╼ $ %f"
}

# Zsh Completions
if [[ "$fpath[(r)${ZSH}/plugins/zsh-completions/src]" ]]; then
  fpath=(${ZSH}/plugins/zsh-completions/src $fpath)
fi

# Zsh Autosuggestions
source $ZSH_CUSTOM/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh

# Zsh Syntax Highlighting
source $ZSH_CUSTOM/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
```
