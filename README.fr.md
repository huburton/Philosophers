# Philosophers

*[English version](README.md)*

Le problème du dîner des philosophes en C, avec des threads POSIX et des mutex. Projet de l'école 19 (réseau 42).

N philosophes sont assis autour d'une table, une fourchette entre chaque paire. Pour manger, il en faut
deux ; ensuite le philosophe dort, puis pense. Le programme doit garder tout le monde en vie tant que les
durées le permettent, annoncer une mort en moins de 10 ms et ne jamais laisser deux threads écrire en
même temps.

```text
$ ./philo 5 800 200 200 3      # personne ne doit mourir, arrêt après 3 repas chacun
...
1600 5 has taken a fork
1600 5 is eating
$ ./philo 1 800 200 200        # une seule fourchette : le philosophe doit mourir
801 1 died
$ ./philo 4 310 200 100        # durées trop serrées : une mort est attendue
311 1 died
```

## Utilisation

```bash
make
./philo nombre_de_philosophes temps_avant_mort temps_pour_manger temps_de_sommeil [repas_par_philosophe]
```

Les durées sont en millisecondes. Chaque ligne du journal suit le format `horodatage_ms id_philosophe action`.

## Fonctionnement

- Un thread par philosophe, un mutex par fourchette (`init.c`, `thread.c`).
- L'ordre de prise des fourchettes dépend de la parité de la place, et les places paires démarrent un peu
  plus tard : l'attente circulaire qui causerait un interblocage ne peut pas se former (`eat_utils.c`,
  `philosopher.c`).
- Un thread de surveillance vérifie l'heure du dernier repas de chacun et le nombre de repas, et arrête la
  simulation (`monitor.c`) ; l'état partagé est protégé par un `death_lock`, et tout affichage passe par un
  `print_lock`.
- Le sommeil passe par une boucle `precise_usleep` sur l'horloge réelle, qui se réveille plus souvent à
  l'approche de l'échéance : les longues attentes ne dérivent pas (`actions.c`).
