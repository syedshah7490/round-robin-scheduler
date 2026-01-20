# round-robin-scheduler
A bash-based Round Robin CPU scheduling simulator with both preemptive and non-preemptive modes. Displays Gantt charts and performance metrics.
#!/bin/bash

# Round Robin Scheduling Simulator
# OS Lab Project

clear
echo "=========================================="
echo "   ROUND ROBIN SCHEDULING SIMULATOR"
echo "=========================================="
echo

# ---------------- GLOBAL ARRAYS ----------------
PID=()
AT=()
BT=()
CT=()
TAT=()
WT=()
GANTT_PID=()
GANTT_TIME=()

QUANTUM=3
NUM_PROCESSES=5

# ---------------- MENU ----------------
get_scheduling_type() {
    echo "Select Scheduling Type:"
    echo "1. Preemptive Round Robin"
    echo "2. Non-Preemptive Round Robin"
    read -p "Enter your choice (1 or 2): " choice
    echo

    if [[ $choice == 1 ]]; then
        SCHED_TYPE="preemptive"
    elif [[ $choice == 2 ]]; then
        SCHED_TYPE="non-preemptive"
    else
        echo "Invalid choice! Defaulting to Preemptive."
        SCHED_TYPE="preemptive"
    fi
}

# ---------------- INPUT ----------------
get_process_details() {
    echo "Time Quantum: $QUANTUM"
    echo "Number of Processes: $NUM_PROCESSES"
    echo

    for ((i=0; i<NUM_PROCESSES; i++)); do
        read -p "Process P$((i+1)) Arrival Time: " at
        read -p "Process P$((i+1)) Burst Time: " bt
        PID+=("P$((i+1))")
        AT+=($at)
        BT+=($bt)
        echo
    done
}

# ---------------- PREEMPTIVE RR ----------------
preemptive_rr() {
    REM_BT=("${BT[@]}")
    ready_queue=()
    current_time=0
    completed=0
    GANTT_TIME=(0)

    while (( completed < NUM_PROCESSES )); do
        for ((i=0; i<NUM_PROCESSES; i++)); do
            if (( AT[i] <= current_time && REM_BT[i] > 0 )); then
                [[ " ${ready_queue[*]} " =~ " $i " ]] || ready_queue+=($i)
            fi
        done

        if (( ${#ready_queue[@]} == 0 )); then
            ((current_time++))
            continue
        fi

        idx=${ready_queue[0]}
        ready_queue=("${ready_queue[@]:1}")

        exec_time=$QUANTUM
        (( REM_BT[idx] < QUANTUM )) && exec_time=${REM_BT[idx]}

        GANTT_PID+=("${PID[idx]}")
        ((current_time += exec_time))
        GANTT_TIME+=($current_time)
        ((REM_BT[idx] -= exec_time))

        if (( REM_BT[idx] == 0 )); then
            CT[idx]=$current_time
            ((completed++))
        else
            ready_queue+=($idx)
        fi
    done

    calculate_metrics
}

# ---------------- NON-PREEMPTIVE RR ----------------
non_preemptive_rr() {
    REM_BT=("${BT[@]}")
    ready_queue=()
    current_time=0
    completed=0
    GANTT_TIME=(0)

    while (( completed < NUM_PROCESSES )); do
        for ((i=0; i<NUM_PROCESSES; i++)); do
            if (( AT[i] <= current_time && REM_BT[i] > 0 )); then
                [[ " ${ready_queue[*]} " =~ " $i " ]] || ready_queue+=($i)
            fi
        done

        if (( ${#ready_queue[@]} == 0 )); then
            ((current_time++))
            continue
        fi

        idx=${ready_queue[0]}
        ready_queue=("${ready_queue[@]:1}")

        GANTT_PID+=("${PID[idx]}")
        ((current_time += REM_BT[idx]))
        GANTT_TIME+=($current_time)

        REM_BT[idx]=0
        CT[idx]=$current_time
        ((completed++))
    done

    calculate_metrics
}

# ---------------- METRICS ----------------
calculate_metrics() {
    total_tat=0
    total_wt=0

    for ((i=0; i<NUM_PROCESSES; i++)); do
        TAT[i]=$((CT[i] - AT[i]))
        WT[i]=$((TAT[i] - BT[i]))
        ((total_tat += TAT[i]))
        ((total_wt += WT[i]))
    done

    display_results
    display_gantt_chart
}

# ---------------- OUTPUT ----------------
display_results() {
    echo
    printf "%-8s %-5s %-5s %-5s %-5s %-5s\n" P AT BT CT TAT WT
    echo "-------------------------------------------"
    for ((i=0; i<NUM_PROCESSES; i++)); do
        printf "%-8s %-5d %-5d %-5d %-5d %-5d\n" \
        "${PID[i]}" "${AT[i]}" "${BT[i]}" "${CT[i]}" "${TAT[i]}" "${WT[i]}"
    done

    echo
    echo "Average TAT: $(echo "scale=2;$total_tat/$NUM_PROCESSES" | bc)"
    echo "Average WT : $(echo "scale=2;$total_wt/$NUM_PROCESSES" | bc)"
}

display_gantt_chart() {
    echo
    echo "GANTT CHART"
    for p in "${GANTT_PID[@]}"; do
        printf "| %s " "$p"
    done
    echo "|"
    printf "%s " "${GANTT_TIME[@]}"
    echo
}

# ---------------- MAIN ----------------
get_scheduling_type
get_process_details

if [[ $SCHED_TYPE == "preemptive" ]]; then
    preemptive_rr
else
    non_preemptive_rr
fi

echo
echo "Thank you for using Round Robin Scheduler!"
