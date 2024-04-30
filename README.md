# Recsys_exploration
Дан датасет кликов ппрослушиваний музыки https://www.kaggle.com/competitions/kkbox-music-recommendation-challenge/data. На его онове необходмио построить рекомендации для пользователей и оценить их, используя метрику NDCG@20. Но так как дадасет был для другого соревнования, то 1 в target, если пользователь в течение месяца переслушал трек еще раз, и 0 в противном случае. Я проинтерпретировал это как 1, если пользователю трек понравился(он его прослушал хотя бы 2 раза), и 0 в случае если нет. Таким образом получается классическая для задач рекомендации таблица с положительными и отрицательными взаимодействиями.

#Подходы к решению:
1) Для каждого пользователя и объекта были получены числовые признаки. После чего данная информация использовалась, как характеристика объектов. Используя эту информацию, с помощью XGBRanker, для каждого пользователя из тестовой части были получены предсказания, и получен скор.
2) Коллаборативная фильтрация. Этот метод не использовал данные объектов, а использовал только матрицу взаимодействий между юзером и айтемом. В этом подходе использовалось 2 модели:
   
  1. kNN. Так как у нас относительно много треков, то был выбран способ рекомендации Item2Item. Для каждого пользователя искались наиболее похожие(близкие по расстоянию) на понравившиеся айтемы. В качестве меры расстояния было использовано косинусное расстояние и евклидово.
  2. Implicit ALS. Реализация факторизации матрицы для получения эмбеддингов для последующей рекомендации по ним.
3) идея получить предсказания, используя подход Item2Vec. Для этого для каждого пользователя была собрана последовательность понравившихся треков. Далее будем воспринимать этот массив, как полследовательность слов, и нам надо предсказать слова дальше(наиболее похожие). Для получения эмбеддингов соответсвенно использовался Word2Vec. Далее рекомендации строились бы по косинусной близости, но это я не успе :(

#Сравнение моделей:
| model_name    | XGBRanker          | kNN (metric='cosine') | kNN (metric='euclidean') | Implicit ALS       |
|---------------|--------------------|-----------------------|--------------------------|--------------------|
| NDCG@20 score | 0.8203252458244168 | 0.6157212998635732    | 0.6544712652880724       | 0.5715601847466224 |

Таким образом, лучше всего себя показал подход, использующий доополнительную информацию о юзерах и айтемах. Значение метрики достаточно большое(max=1.0). Поэтому можно сказать, что, в среднем, он очень хорошо справляется с рекомендацией треков. Но нельзя забывать о проблеме холодного старта. Для таких пользователей очень трудно что-то порекомендовать, и метрика из-за них уменьшается(аналогично с пользователями с маленьким взаимодействием).
