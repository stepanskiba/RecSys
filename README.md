Для решения задачи рекомендации на основе MovieLens я выбрал библиотеку Pytorch и подход NCF. Мы будем тестироваться на рекомендации(подборке) 10 фильмов для каждого пользователя.Данные для обучения я предварительно загрузил на гугл диск для удобного доступа из colab. Далее, в классе UserItemRatingDataset я сделал представление наших данных в виде torch.tensor.

Я решил перевести Excplicit feedback в Implicit(т.е. оценки пользователями фильмов в бинарную метрику поставил оценку/не поставил), обычно Implicit feedbackа гораздо больше, поэтому было бы разумно сразу обучать модель так. В классе NCFData я создаю данные для обучения/тестирования. Важно отметить, что нам нужно вручную создать для пользователей отрицательные примеры(сейчас мы знаем только про положительные, т.е. оценки от пользователей). Для этого случайно будет добавлять к пользователем фильмы, которые он не оценил. Для тестирования будем использовать подход Leave One Out, то есть тестировать на основе последней оценки фильма от пользователя.

К сожалению, при полной загрузке датасета MovieLens 25m вылетает оперативная память, так что я ограничусь примерно 500к оценок(если необходимо, при больших вычислительных мощностях можно засунуть и полный датасет).

Для оценки качества модели я выбрал метрики Hit(попадание релевантного элемента в рекомендуемые, без учета их ранжирования) и nDCG(normalized Discounted Cumulative Gain), где уже учитывается ранжирование(т.е. более релевантные элементы должны быть выше в списке рекомендаций).

Далее я реализовал модель GMF(матричное разложение матрицы пользователей и фильмов). С помощью nn.Embeddings мы создаем два эмбеддинга пользователей и фильмов, перемножаем их поэлементно и пропускаем через полносвязный слой с активацией сигмоида.

В конце, обучаем нашу нейронную сеть с помощью train_pipeline и смотрим на средние значения метрик Hit и nDCG на каждой эпохе. Уже после нескольких эпох мы достигаем неплохо качества Hit и nDCG(порядка 0.8 и 0.5 уже на 2 эпохе). Это достаточно хорошо, так как получается для 80% пользователей мы смогли порекомендовать такие фильмы, что они посмотрели и оценили хотя-бы один из них.

