#!/bin/bash

EDITOR="${EDITOR:-nano}"
NUMCOLOR="\e[38;5;82m"       # green for line numbers
COLOR="${COLOR:-\e[38;5;208m}" # orange for paths
RESET="\e[0m"

# Priority configs to show by default
PRIORITY=(
"/etc/postfix/main.cf"
"/etc/fail2ban/jail.local"
"/etc/apache2/apache2.conf"
"/etc/dovecot/dovecot.conf"
"/etc/ssh/sshd_config"
"/etc/mysql/mysql.conf.d/mysqld.cnf"
"/etc/php/*/apache2/php.ini"
"/etc/ufw/ufw.conf"
"/etc/systemd/system.conf"
)

# Function to find all config files
find_configs() {
    find /etc -type f \( \
        -name "*.conf" \
        -o -name "*.ini" \
        -o -name "*.cf" \
        -o -name "*.local" \
        -o -name "*.cnf" \
        -o -name "*.config" \
        -o -name "*.yaml" \
        -o -name "*.yml" \
        -o -name "*.json" \
        -o -name "*.env" \
        -o -name "*.rules" \
        -o -name "*.list" \
        -o -name "*.service" \
        -o -name "*.socket" \
        -o -name "*.timer" \
    \)
}

# Determine which list to use
if [ "$1" == "-all" ]; then
    CONFIGS=( $(find_configs | sort) )
elif [ "$1" == "-q" ] && [ -n "$2" ]; then
    CONFIGS=( $(find /etc -type f -name "*$2" | sort) )
else
    CONFIGS=()
    for f in "${PRIORITY[@]}"; do
        for file in $f; do
            [ -f "$file" ] && CONFIGS+=("$file")
        done
    done
fi


count=${#CONFIGS[@]}
[ $count -eq 0 ] && { echo "No configs found."; exit 1; }

# Determine terminal width and column layout
term_width=$(tput cols)
max_len=0
for f in "${CONFIGS[@]}"; do
    (( ${#f} > max_len )) && max_len=${#f}
done
col_width=$((max_len + 6))   # number + padding
cols=$(( term_width / col_width ))
[ $cols -lt 1 ] && cols=1
rows=$(( (count + cols - 1) / cols ))

# Compute dynamic width for each column
declare -a col_widths
for ((c=0; c<cols; c++)); do
    max=0
    for ((r=0; r<rows; r++)); do
        idx=$(( r*cols + c ))
        [ $idx -lt $count ] || continue
        len=${#CONFIGS[$idx]}
        (( len > max )) && max=$len
    done
    col_widths[$c]=$((max + 6))
done

# Display configs row-wise with proper numbering
echo -e "${COLOR}Select a config to edit:${RESET}"
num=1
for ((r=0; r<rows; r++)); do
    line=""
    for ((c=0; c<cols; c++)); do
        idx=$(( r*cols + c ))
        [ $idx -lt $count ] || continue
        width=${col_widths[$c]}
        line="$line $(printf "${NUMCOLOR}%3d)${RESET} %-*s" $num $width "${CONFIGS[$idx]}")"
        ((num++))
    done
    echo -e "$line"
done

# Ask for user input and open editor
read -p "Enter number: " choice
if [[ "$choice" =~ ^[0-9]+$ ]] && [ "$choice" -ge 1 ] && [ "$choice" -le "$count" ]; then
    "$EDITOR" "${CONFIGS[$((choice-1))]}"
fi
