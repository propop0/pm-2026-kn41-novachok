# Прогноз: метод і висновки

Студент: Новачок Богдан (propop0). Тема: Телеграм-бот деканату: довідки, черга на консультації, оголошення. Варіант вхідних умов: V06.

## 1. Звідки взялася velocity

| Поле | Значення |
| --- | --- |
| Спринтів у `velocity.csv` | 2 (Спринт 1 закрито 2026-10-22: узято 20, закрито 17 points; Спринт 2 триває з 2026-10-23 по 2026-11-05: узято 18 points) |
| Тривалість спринта | два тижні (14 календарних / 10 робочих днів) |
| Ємність спринта з мого варіанта | 20 points |
| Середня фактична velocity | розраховується після закриття Спринта 2 (2026-11-05) |
| Найгірший і найкращий спринт | розраховується після закриття Спринта 2 (2026-11-05) |

Звідки бралися points закритих історій: Усі числа беруться виключно зі стовпчика `final_estimate` файла `lr08_poker/estimates.csv` (у Спринті 1 закрито 5 історій першого релізу: `S-01` = 2, `S-09` = 5, `S-03` = 3, `S-04` = 5, `S-05` = 2, що в сумі дає 17 points; історії поза `estimates.csv` у спринти не бралися).

Чому факт розійшовся з ємністю варіанта: У Спринті 1 команда закрила 17 points проти номінальної ємності варіанта 20 points (відхилення 15%, у межах норми третини), оскільки історія `S-02` («Профіль студента», 3 points) простояла 3 дні у блоці через виявлені дублікати шифрів, порожні поля та архівні записи у старих Excel-таблицях груп деканату і була перенесена в Спринт 2.

## 2. Як рахувалася симуляція

| Поле | Значення |
| --- | --- |
| Інструмент | власний скрипт Python (`random.Random(42)`, стандартна бібліотека `random` і `math`) |
| Прогонів | 10000 |
| Що тягнеться випадково | фактична velocity (`points_done`) одного зі спринтів історії з рівною імовірністю (`rng.choice(velocities)`) |
| Як рахується дата прогнозу | останній день спринта, на якому закінчується робота: для сценарію «Весь перший реліз» кінець Спринта 1 (`2026-10-22`) плюс `(P − 1) × 14` днів; для сценарію «Залишок після двох спринтів» кінець Спринта 2 (`2026-11-05`) плюс `P × 14` днів |

```python
import csv
import math
import random


def simulate_monte_carlo(velocities, remaining_points, simulations=10000, seed=42):
    rng = random.Random(seed)
    sprints_needed = []
    for _ in range(simulations):
        rem = remaining_points
        sprints = 0
        while rem > 0:
            rem -= rng.choice(velocities)
            sprints += 1
        sprints_needed.append(sprints)
    sprints_needed.sort()
    p50 = sprints_needed[math.ceil(0.50 * simulations) - 1]
    p85 = sprints_needed[math.ceil(0.85 * simulations) - 1]
    return p50, p85


with open('lr09_reference_velocity.csv', encoding='utf-8') as f:
    rows = list(csv.DictReader(f))

v_all = [int(r['points_done']) for r in rows]
v_1_2 = [int(r['points_done']) for r in rows if int(r['sprint']) in (1, 2)]
v_5_6 = [int(r['points_done']) for r in rows if int(r['sprint']) in (5, 6)]

for name, v in [('Усі 12 спринтів', v_all), ('Спринти 1 і 2', v_1_2), ('Спринти 5 і 6', v_5_6)]:
    p50, p85 = simulate_monte_carlo(v, remaining_points=220, simulations=10000, seed=42)
    print(f'{name}: P50={p50}, P85={p85}, інтервал={p85 - p50}')
