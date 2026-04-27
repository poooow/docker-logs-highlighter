# Docker logs highlight

Insert into your `~/.bashrc` 
then run `source ~/.bashrc` to apply

```bash
# Highlight docker logs
docker() {
    local is_logs=0
    for arg in "$@"; do
        [[ "$arg" == "logs" ]] && is_logs=1 && break
        [[ "$arg" != -* ]] && break
    done

    if [[ $is_logs -eq 1 ]]; then
        command docker "$@" 2>&1 | while IFS= read -r line; do

            # Keyword badge colors
            local RESET=$'\033[0m'
            local BADGE_ERROR=$'\033[1;37;41m'
            local BADGE_WARN=$'\033[1;30;43m'
            local BADGE_INFO=$'\033[1;37;42m'
            local BADGE_DEBUG=$'\033[1;37;44m'
            local BADGE_TRACE=$'\033[1;37;46m'
            local BADGE_EXCEPT=$'\033[1;37;45m'

            # Pick line tint
            local R=''
            if echo "$line" | grep -qiE '(FATAL|CRITICAL|ERROR|ERR)'; then
                R=$'\033[0;31m'
            elif echo "$line" | grep -qiE '(WARN(ING)?)'; then
                R=$'\033[0;33m'
            elif echo "$line" | grep -qiE '\bINFO\b'; then
                R=$'\033[0;32m'
            elif echo "$line" | grep -qiE '\bDEBUG\b'; then
                R=$'\033[0;34m'
            elif echo "$line" | grep -qiE '(TRACE|VERBOSE)'; then
                R=$'\033[0;36m'
            elif echo "$line" | grep -qiE '(Exception|Traceback)'; then
                R=$'\033[0;35m'
            fi

            # Highlight keywords, restore line tint after each badge
            line=$(echo "$line" | sed \
                -e "s/FATAL/${BADGE_ERROR}FATAL${R}/gI" \
                -e "s/CRITICAL/${BADGE_ERROR}CRITICAL${R}/gI" \
                -e "s/\bERROR\b/${BADGE_ERROR}ERROR${R}/gI" \
                -e "s/\bERR\b/${BADGE_ERROR}ERR${R}/gI" \
                -e "s/\bWARNING\b/${BADGE_WARN}WARNING${R}/gI" \
                -e "s/\bWARN\b/${BADGE_WARN}WARN${R}/gI" \
                -e "s/\bINFO\b/${BADGE_INFO}INFO${R}/gI" \
                -e "s/\bDEBUG\b/${BADGE_DEBUG}DEBUG${R}/gI" \
                -e "s/\bTRACE\b/${BADGE_TRACE}TRACE${R}/gI" \
                -e "s/\bVERBOSE\b/${BADGE_TRACE}VERBOSE${R}/gI" \
                -e "s/Exception/${BADGE_EXCEPT}Exception${R}/g" \
                -e "s/Traceback/${BADGE_EXCEPT}Traceback${R}/g" \
            )

            # Wrap line in tint color
            if [[ -n "$R" ]]; then
                echo "${R}${line}${RESET}"
            else
                echo "$line"
            fi

        done
    else
        command docker "$@"
    fi
}
```
