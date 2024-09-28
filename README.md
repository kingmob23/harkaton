Общее описание подхода: мы объединяем фильтрацию на основе содержания (Ontent-Based Filtering) с кластеризацией (Clustering) и коллаборативную фильтрацию (Collaborative Filtering) для предоставления персонализированных видеорекомендаций с учетом компромисса между исследованием и эксплуатацией (Explore Exploit tradeoff).

~~Reinforcement Learning (RL) at the Top Level Exploration-Exploitation is Part of the RL Policy. Multi-armed bandit problem as more simple RL version.~~

RL это про хранения стейта (истории выборов) и про динамическую адаптацию. Пока под вопросом.

На вход в алгоритм мы получаем 10 видео с реакциями на них.
Будет 10 - 20 итераций.

Метрики видео разделим на две группы - для ранжирования и для кластеризации.

1. Метрики для ранжирования (где можно сказать лучше/хуже):
    - v_year_views
    - v_month_views
    - v_week_views
    - v_day_views
    - v_likes
    - v_dislikes
    - v_avg_watchtime_1_day
    - v_avg_watchtime_7_day
    - v_avg_watchtime_30_day
    - v_category_popularity_percent_7_days
    - v_category_popularity_percent_30_days
    - v_pub_datetime (свежее - лучше)
    - v_long_views_1_days (это и далее - предположительно. нужно точно понять что это.)
    - v_long_views_7_days
    - v_long_views_30_days
2. Метрики для кластеризации (где лучше/хуже определить сложно):
    - title
    - description
    - category_id
    - author_id
    - v_duration
    - v_total_comments
3. Не кластеризуемые:
    - row_number
    - video_id
4. Удалить из выдачи:
    - v_is_hidden
    - v_is_deleted

Из метрик для ранжирования получаем единый топ. Для этого каждую из метрик нормализуем, присваиваем вес, вычисляем взвешенную сумму.

Кластеризовать будем с MeanShift.

One effective way to choose the best features for clustering is to identify which numerical features exhibit the highest variance.

Для кластеризации нужно преобразовать каждую метрику в численные значения. Для категорий используем One-Hot Encoding. Он преобразует каждую категорию в бинарный вектор (длинной в кол-во метрик), для каждой записи категория будет представлена единичкой в соответствующей позиции.

Векторы для кажой из метрик объединить в одну матрицу, используя конкатенацию.

---

Для рекомендаций видео будем использовать Content-based filtering algorithms

---

1. Общий подход к рекомендациям:
На верхнем уровне анализируется поведение пользователя, и система стремится найти баланс между исследованием новых интересов (explore) и использованием известных предпочтений (exploit).
Изначально предлагается использовать 100% explore, постепенно снижая этот процент до 50% explore / 50% exploit.
Первые 10 видео - эксплор. (Демографическую по пользователю рутуб получает после первого просмотра, в нашем случае после первого обновления выдачи.)

user_explore_coef = 0

user_rec = user_based(user_explore_coef)

video_rec = recomend_vids(10 - user_explore_coef)

final_rec = user_rec + video_rec


2. Данные пользователя и видео:
Используются [опционально добавим потом - данные пользователя (регион, город) и] метрики видео (v_pub_datetime, v_total_comments, v_year_views, description, category_id, author_id) для более точного подбора контента.


all_clusters = []

checked_clusters = []

def recomend_vids(how_many: int):
    available_clusters = [cluster for cluster in all_clusters is cluster not in checked_clusters]

    vids = [cluster[0] for cluster in available_clusters[:how_many]]

    return vids

4. Учет пользовательских предпочтений:
Если видео получают позитивные оценки (лайк),то этот кластер в следующих рекомендациях учитывается как эксплойтовый.
Негативно оцененные кластеры удаляются из пула для будущих рекомендаций.
5. Построение портрета пользователя:
С ростом информации о пользователе (например, лайки, просмотры) увеличивается роль exploit-системы пользователя.
Постепенно система фокусируется больше на кластерах, которые соответствуют портрету пользователя.
6. Весовые коэффициенты:
В системе может использоваться механизм весов для разных факторов — сначала приоритет отдается видео-метрикам, затем со временем большее влияние начинают оказывать данные о поведении пользователя.
7. Итоговая цель:
Задача системы — сбалансировать отношения между разными уровнями вложенности и метриками, чтобы найти оптимальный баланс между explore и exploit, улучшая рекомендации.

---

Коллаборативная фильтрация — это подход к рекомендациям, основанный на поведении пользователей и взаимодействии с контентом. В твоем случае, если о пользователе нет информации, ты можешь использовать следующие стратегии:

### 1. **Матрица взаимодействий**
Создай матрицу взаимодействий, где строки будут представлять пользователей (или анонимные пользовательские идентификаторы), а столбцы — видео. Ячейки будут содержать значения, отражающие взаимодействие пользователя с видео (например, количество просмотров, лайков, комментариев).

### 2. **Группировка по метаданным**
Если у тебя нет информации о конкретных пользователях, можно использовать метаданные видео для создания более обобщенной модели. Например, можно строить матрицы на основе:
- **category_id**: какие категории видео пользователи чаще смотрят.
- **author_id**: какие авторы наиболее популярны.

### 3. **Анонимная коллаборативная фильтрация**
Можно применить методы коллаборативной фильтрации, не зависящие от конкретных пользователей. Например:
- **Item-based Collaborative Filtering**: анализируй, какие видео смотрят вместе. Если пользователи, которые смотрят видео A, также часто смотрят видео B, это может означать, что B может быть интересным и для других, кто уже смотрел A.
- **Matrix Factorization**: методы, такие как SVD (сингулярное разложение матрицы), могут быть использованы для нахождения скрытых факторов, объясняющих взаимодействия между пользователями и видео.

### 4. **Популярность видео**
Если ты не можешь собрать информацию о пользователях, можно просто рекомендовать видео, которые получили наибольшее количество просмотров или положительных взаимодействий (например, лайков) за определённый период. Это можно комбинировать с твоим методом кластеризации, предлагая топовые видео из каждого кластера.

### Пример реализации
1. **Создание матрицы взаимодействий**:
   - Заполни ячейки, где для каждого видео указано количество просмотров или лайков.

2. **Кластеризация**:
   - Кластеризуй видео по category_id и author_id, как планировал.

3. **Рекомендации**:
   - Выбирай топовые видео по количеству просмотров из каждого кластера и комбинируй их с популярными видео (например, 5 из кластеров + 5 из самых популярных видео).

---

Этот кейс вполне можно рассматривать в контексте задачи многорукого бандита (Multi-armed Bandit Problem). Вот почему:

1. **Неопределенность предпочтений**: У вас есть новые пользователи, о которых нет информации, и вам нужно выяснить, какие видео им интересны. Это соответствует основной задаче многорукого бандита — оптимизации выбора среди нескольких опций, не зная заранее, какая из них наиболее полезна.

2. **Обратная связь**: Бинарная обратная связь (положительная или отрицательная) от пользователей о видео позволяет вам обновлять свою стратегию. Многорукие бандиты прекрасно подходят для случаев, когда вы получаете "награды" (в вашем случае — положительная/отрицательная обратная связь) за выбор контента.

3. **Адаптивность**: Алгоритм может адаптироваться по мере накопления данных. Вы можете использовать различные стратегии, такие как ε-greedy или UCB (Upper Confidence Bound), чтобы постепенно исследовать и использовать видео на платформе, основываясь на полученной информации.

4. **Цель максимизации вовлеченности**: Ваша задача — максимизировать интерес и удержание пользователей, что также связано с темой многорукого бандита, где цель состоит в максимизации общего вознаграждения.

Таким образом, применение подхода многорукого бандита может быть весьма полезным для решения проблемы "холодного старта" в вашей системе рекомендаций.