# Philosophers

*[Version française](README.fr.md)*

The dining philosophers problem in C with POSIX threads and mutexes. School 19 (42 network) project.

N philosophers sit around a table with one fork between each pair. A philosopher needs both forks to eat,
then sleeps, then thinks. The program must keep everyone alive as long as the timings allow it, report a
death within 10 ms, and never let two threads print at the same time.

```text
$ ./philo 5 800 200 200 3      # nobody should die, stops after 3 meals each
...
1600 5 has taken a fork
1600 5 is eating
$ ./philo 1 800 200 200        # one fork only: the philosopher must die
801 1 died
$ ./philo 4 310 200 100        # timings too tight: a death is expected
311 1 died
```

## Usage

```bash
make
./philo number_of_philosophers time_to_die time_to_eat time_to_sleep [meals_per_philosopher]
```

Times are in milliseconds. Each log line is `timestamp_ms philosopher_id action`.

## How it works

- One thread per philosopher, one mutex per fork (`init.c`, `thread.c`).
- Forks are taken in an order that depends on the seat parity, and even seats start slightly later,
  which breaks the circular wait that would cause a deadlock (`eat_utils.c`, `philosopher.c`).
- A monitor thread checks every philosopher's last meal time and the meal count, and stops the simulation
  (`monitor.c`); shared state is protected by a `death_lock`, and all output goes through a `print_lock`.
- Sleeping uses a `precise_usleep` loop on the real clock that wakes up more often near the deadline,
  so long waits do not drift (`actions.c`).
