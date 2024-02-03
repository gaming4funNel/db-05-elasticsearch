# Домашнее задание к занятию 5. «Elasticsearch»

## Задача 1
sdagdsfgdsfgdsgsdfgsdgfsd
В этом задании вы потренируетесь в:

- установке Elasticsearch,
- первоначальном конфигурировании Elasticsearch,
- запуске Elasticsearch в Docker.

Используя Docker-образ [centos:7](https://hub.docker.com/_/centos) как базовый и 
[документацию по установке и запуску Elastcisearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html):

- составьте Dockerfile-манифест для Elasticsearch,
- соберите Docker-образ и сделайте `push` в ваш docker.io-репозиторий,
- запустите контейнер из получившегося образа и выполните запрос пути `/` c хост-машины.

Требования к `elasticsearch.yml`:

- данные `path` должны сохраняться в `/var/lib`,
- имя ноды должно быть `netology_test`.

В ответе приведите:

- текст Dockerfile-манифеста,

```
FROM centos:7.9.2009

RUN yum install perl-Digest-SHA -y

WORKDIR /opt/elasticsearch

COPY elasticsearch-8.11.3-linux-x86_64.tar.gz /opt/elasticsearch
COPY elasticsearch-8.11.3-linux-x86_64.tar.gz.sha512 /opt/elasticsearch

RUN shasum -a 512 -c elasticsearch-8.11.3-linux-x86_64.tar.gz.sha512
RUN tar -xzf elasticsearch-8.11.3-linux-x86_64.tar.gz
RUN rm elasticsearch-8.11.3-linux-x86_64.tar.gz

COPY elasticsearch.yml /opt/elasticsearch/elasticsearch-8.11.3/config

RUN useradd elasticsearch

RUN mkdir -p "/var/lib/elasticsearch"
RUN mkdir -p "/var/log/elasticsearch"

RUN chown -R elasticsearch:elasticsearch /opt/elasticsearch
RUN chown -R elasticsearch:elasticsearch /var/lib/elasticsearch
RUN chown -R elasticsearch:elasticsearch /var/log/elasticsearch

USER elasticsearch

WORKDIR /opt/elasticsearch/elasticsearch-8.11.3

ENTRYPOINT ["./bin/elasticsearch"]
```

- ссылку на образ в репозитории dockerhub,

https://hub.docker.com/layers/gaming4funnel/elasticsearch/latest/images/sha256-191bf9374405a22390d18b3ded035feb4b133716a5bee549ad57e93376237cca?context=repo

- ответ `Elasticsearch` на запрос пути `/` в json-виде.

```
root@dbpostgres:/opt/elastic# curl -X GET "localhost:9200/"
{
  "name" : "netology_test",
  "cluster_name" : "netology",
  "cluster_uuid" : "RAWf-zopTEa-_ej2WVCnLA",
  "version" : {
    "number" : "8.11.3",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "64cf052f3b56b1fd4449f5454cb88aca7e739d9a",
    "build_date" : "2023-12-08T11:33:53.634979452Z",
    "build_snapshot" : false,
    "lucene_version" : "9.8.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

Подсказки:

- возможно, вам понадобится установка пакета perl-Digest-SHA для корректной работы пакета shasum,
- при сетевых проблемах внимательно изучите кластерные и сетевые настройки в elasticsearch.yml,
- при некоторых проблемах вам поможет Docker-директива ulimit,
- Elasticsearch в логах обычно описывает проблему и пути её решения.

Далее мы будем работать с этим экземпляром Elasticsearch.

## Задача 2

В этом задании вы научитесь:

- создавать и удалять индексы,
- изучать состояние кластера,
- обосновывать причину деградации доступности данных.

Ознакомьтесь с [документацией](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html) 
и добавьте в `Elasticsearch` 3 индекса в соответствии с таблицей:

| Имя | Количество реплик | Количество шард |
|-----|-------------------|-----------------|
| ind-1| 0 | 1 |
| ind-2 | 1 | 2 |
| ind-3 | 2 | 4 |

Получите список индексов и их статусов, используя API, и **приведите в ответе** на задание.

![elastic](https://github.com/gaming4funNel/db-05-elasticsearch/blob/main/img/elastic1.png)

Получите состояние кластера `Elasticsearch`, используя API.

![elastic](https://github.com/gaming4funNel/db-05-elasticsearch/blob/main/img/elastic2.png)

Как вы думаете, почему часть индексов и кластер находятся в состоянии yellow?

Потому что указано количество реплик, но в кластере только одна нода, из-за этого по двум индексам у нас появились нераспределенные шарды.

Удалите все индексы.

![elastic](https://github.com/gaming4funNel/db-05-elasticsearch/blob/main/img/elastic3.png)

**Важно**

При проектировании кластера Elasticsearch нужно корректно рассчитывать количество реплик и шард,
иначе возможна потеря данных индексов, вплоть до полной, при деградации системы.

## Задача 3

В этом задании вы научитесь:

- создавать бэкапы данных,
- восстанавливать индексы из бэкапов.

Создайте директорию `{путь до корневой директории с Elasticsearch в образе}/snapshots`.

Используя API, [зарегистрируйте](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-register-repository.html#snapshots-register-repository) 
эту директорию как `snapshot repository` c именем `netology_backup`.

**Приведите в ответе** запрос API и результат вызова API для создания репозитория.

![elastic](https://github.com/gaming4funNel/db-05-elasticsearch/blob/main/img/elastic4.png)

Создайте индекс `test` с 0 реплик и 1 шардом и **приведите в ответе** список индексов.

![elastic](https://github.com/gaming4funNel/db-05-elasticsearch/blob/main/img/elastic5.png)

[Создайте `snapshot`](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html) 
состояния кластера `Elasticsearch`.

**Приведите в ответе** список файлов в директории со `snapshot`.

![elastic](https://github.com/gaming4funNel/db-05-elasticsearch/blob/main/img/elastic6.png)

Удалите индекс `test` и создайте индекс `test-2`. **Приведите в ответе** список индексов.

![elastic](https://github.com/gaming4funNel/db-05-elasticsearch/blob/main/img/elastic7.png)

[Восстановите](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-restore-snapshot.html) состояние
кластера `Elasticsearch` из `snapshot`, созданного ранее. 

**Приведите в ответе** запрос к API восстановления и итоговый список индексов.

![elastic](https://github.com/gaming4funNel/db-05-elasticsearch/blob/main/img/elastic8.png)

Подсказки:

- возможно, вам понадобится доработать `elasticsearch.yml` в части директивы `path.repo` и перезапустить `Elasticsearch`.

---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---