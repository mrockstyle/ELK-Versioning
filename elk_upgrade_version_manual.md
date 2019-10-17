# ELK Versioning for sumo nonprd cluster
## Table of Contents
1. [Prerequisite](#1-prerequisite)
2. [Upgrade elasticsearch 5.5.2 => 5.6](#2-upgrade-elasticsearch-from-version-552--56)
3. [Upgrade elasticsearch 5.6 => 6.8.0](#3-upgrade-elasticsearch-from-version-56--680)
4. [Upgrade elasticsearch 6.8.0 => 7.3.1](#4-upgrade-elasticsearch-from-version-680--731)
5. [Upgrade Kibana 5.5.2 => 7.3.1](#5-kibana)
6. [Enable cron at APP VM](#6-enable-cronjob-at-app-vm)

---

## 1. Prerequisite

### 1.1 Download ***elk_versioning_packages.tar.gz***
### Inside elk_versioning_packages.tar.gz
- docker-compose.yml //for elasticsearch x.x.x
- docker-compose-kib-7_3.yml //for kibana 7.3.1
- elasticsearch_config
- kibana_config
- elk_env //version control setting

### 1.2 Preparation
> Host: App, DB
```bash
cd /local/elasticsearch
tar xzvf elk_versioning_packages.tar.gz

# App VM
cp -p kibana_config/* /local/elasticsearch/kibana/config/

# DB VM
cp -p elasticsearch_config/elasticsearch1/* /local/elasticsearch/elasticsearch1/config/
cp -p elasticsearch_config/elasticsearch2/* /local/elasticsearch/elasticsearch2/config/
```

### 1.3 Disable ingest node cronjob at APP VM
> Host: App
```bash
# DEV
mv /var/spool/cron/crontabs/devadm /var/spool/cron/crontabs/devadm.disabled

# SIT & UAT
mv /var/spool/cron/crontabs/dockadm /var/spool/cron/crontabs/dockadm.disabled
```

### 1.4 Stop ELK Services (Elasticsearch 5.5.2, Kibana 5.5.2)
> Host: App, DB
```bash
docker-compose-cluster down
```
---

## 2. UPGRADE ELASTICSEARCH FROM VERSION 5.5.2 => 5.6
> Host: DB
### 2.1 Backup data
```bash
cp -r -p /local/elasticsearch/elasticsearch1/data /local/elasticsearch/elasticsearch1/data5_5.bak
cp -r -p /local/elasticsearch/elasticsearch2/data /local/elasticsearch/elasticsearch2/data5_5.bak
```

### 2.2 Declare elasticsearch environment
```bash
. ./elk_env
```

### 2.3 Start version 5.6
```bash
docker-compose up -d
```

---

## 3. UPGRADE ELASTICSEARCH FROM VERSION 5.6 => 6.8.0
> Host: DB
### 3.1 Backup data
```bash
cp -r -p /local/elasticsearch/elasticsearch1/data /local/elasticsearch/elasticsearch1/data5_6.bak
cp -r -p /local/elasticsearch/elasticsearch2/data /local/elasticsearch/elasticsearch2/data5_6.bak
```

### 3.2 Stop version 5.6
```bash
docker-compose down
```

### 3.3 Declare elasticsearch environment
```bash
sed -i 's/5.6/6.8.0/' elk_env
. ./elk_env
```

### 3.4 Patch Permission
```bash
chmod -R 777 /local/elasticsearch/elasticsearch1/data
chmod 777 /local/elasticsearch/elasticsearch1/logs

chmod -R 777 /local/elasticsearch/elasticsearch2/data
chmod 777 /local/elasticsearch/elasticsearch2/logs
```

### 3.5 Start version 6.8.0
```bash
docker-compose up -d
```

---

## 4. UPGRADE ELASTICSEARCH FROM VERSION 6.8.0 => 7.3.1
> Host: DB
### 4.1 Backup data
```bash
cp -r -p /local/elasticsearch/elasticsearch1/data /local/elasticsearch/elasticsearch1/data6_8.bak
cp -r -p /local/elasticsearch/elasticsearch2/data /local/elasticsearch/elasticsearch2/data6_8.bak
```

### 4.2 Kibana Reindex
```bash
# Set .kibana index to read-only

curl -X PUT "localhost:9200/.kibana/_settings?pretty" -H 'Content-Type: application/json' -d'
{
  "index.blocks.write": true
}
'
```

```bash
# Create .kibana-6 index

curl -X PUT "localhost:9200/.kibana-6?pretty" -H 'Content-Type: application/json' -d'
{
  "settings" : {
    "number_of_shards" : 1,
    "index.mapper.dynamic": false
  },
  "mappings" : {
    "doc": {
      "properties": {
        "type": {
          "type": "keyword"
        },
        "updated_at": {
          "type": "date"
        },
        "config": {
          "properties": {
            "buildNum": {
              "type": "keyword"
            }
          }
        },
        "index-pattern": {
          "properties": {
            "fieldFormatMap": {
              "type": "text"
            },
            "fields": {
              "type": "text"
            },
            "intervalName": {
              "type": "keyword"
            },
            "notExpandable": {
              "type": "boolean"
            },
            "sourceFilters": {
              "type": "text"
            },
            "timeFieldName": {
              "type": "keyword"
            },
            "title": {
              "type": "text"
            }
          }
        },
        "visualization": {
          "properties": {
            "description": {
              "type": "text"
            },
            "kibanaSavedObjectMeta": {
              "properties": {
                "searchSourceJSON": {
                  "type": "text"
                }
              }
            },
            "savedSearchId": {
              "type": "keyword"
            },
            "title": {
              "type": "text"
            },
            "uiStateJSON": {
              "type": "text"
            },
            "version": {
              "type": "integer"
            },
            "visState": {
              "type": "text"
            }
          }
        },
        "search": {
          "properties": {
            "columns": {
              "type": "keyword"
            },
            "description": {
              "type": "text"
            },
            "hits": {
              "type": "integer"
            },
            "kibanaSavedObjectMeta": {
              "properties": {
                "searchSourceJSON": {
                  "type": "text"
                }
              }
            },
            "sort": {
              "type": "keyword"
            },
            "title": {
              "type": "text"
            },
            "version": {
              "type": "integer"
            }
          }
        },
        "dashboard": {
          "properties": {
            "description": {
              "type": "text"
            },
            "hits": {
              "type": "integer"
            },
            "kibanaSavedObjectMeta": {
              "properties": {
                "searchSourceJSON": {
                  "type": "text"
                }
              }
            },
            "optionsJSON": {
              "type": "text"
            },
            "panelsJSON": {
              "type": "text"
            },
            "refreshInterval": {
              "properties": {
                "display": {
                  "type": "keyword"
                },
                "pause": {
                  "type": "boolean"
                },
                "section": {
                  "type": "integer"
                },
                "value": {
                  "type": "integer"
                }
              }
            },
            "timeFrom": {
              "type": "keyword"
            },
            "timeRestore": {
              "type": "boolean"
            },
            "timeTo": {
              "type": "keyword"
            },
            "title": {
              "type": "text"
            },
            "uiStateJSON": {
              "type": "text"
            },
            "version": {
              "type": "integer"
            }
          }
        },
        "url": {
          "properties": {
            "accessCount": {
              "type": "long"
            },
            "accessDate": {
              "type": "date"
            },
            "createDate": {
              "type": "date"
            },
            "url": {
              "type": "text",
              "fields": {
                "keyword": {
                  "type": "keyword",
                  "ignore_above": 2048
                }
              }
            }
          }
        },
        "server": {
          "properties": {
            "uuid": {
              "type": "keyword"
            }
          }
        },
        "timelion-sheet": {
          "properties": {
            "description": {
              "type": "text"
            },
            "hits": {
              "type": "integer"
            },
            "kibanaSavedObjectMeta": {
              "properties": {
                "searchSourceJSON": {
                  "type": "text"
                }
              }
            },
            "timelion_chart_height": {
              "type": "integer"
            },
            "timelion_columns": {
              "type": "integer"
            },
            "timelion_interval": {
              "type": "keyword"
            },
            "timelion_other_interval": {
              "type": "keyword"
            },
            "timelion_rows": {
              "type": "integer"
            },
            "timelion_sheet": {
              "type": "text"
            },
            "title": {
              "type": "text"
            },
            "version": {
              "type": "integer"
            }
          }
        },
        "graph-workspace": {
          "properties": {
            "description": {
              "type": "text"
            },
            "kibanaSavedObjectMeta": {
              "properties": {
                "searchSourceJSON": {
                  "type": "text"
                }
              }
            },
            "numLinks": {
              "type": "integer"
            },
            "numVertices": {
              "type": "integer"
            },
            "title": {
              "type": "text"
            },
            "version": {
              "type": "integer"
            },
            "wsState": {
              "type": "text"
            }
          }
        }
      }
    }
  }
}
'
```

```bash
# Reindex .kibana into .kibana-6

curl -X POST "localhost:9200/_reindex?pretty" -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": ".kibana"
  },
  "dest": {
    "index": ".kibana-6"
  },
  "script": {
    "inline": "ctx._source = [ ctx._type : ctx._source ]; ctx._source.type = ctx._type; ctx._id = ctx._type + \":\" + ctx._id; ctx._type = \"doc\"; ",
    "lang": "painless"
  }
}
'
```

```bash
# Alias .kibana-6 to .kibana and remove legacy .kibana index

curl -X POST "localhost:9200/_aliases?pretty" -H 'Content-Type: application/json' -d'
{
  "actions" : [
    { "add":  { "index": ".kibana-6", "alias": ".kibana" } },
    { "remove_index": { "index": ".kibana" } }
  ]
}
'
```

## 4.3 Reindex

```bash
# reindex
for index in $(curl -X GET "localhost:9200/_cat/indices?v&pretty" | awk '{print $3}' | grep -vw "index" | grep -v "kibana")
do
  curl -HContent-Type:application/json -XPOST localhost:9200/_reindex?pretty -d'{
    "source": {
      "index": "'$index'"
    },
    "dest": {
      "index": "'$index'-reindexed"
    }
  }'
done
```


```bash
# Set Alias

for index in $(curl -X GET "localhost:9200/_cat/indices?v&pretty" | awk '{print $3}' | grep -vw "index" | grep -v "kibana" | grep -v "reindexed")
do
    curl -X POST "localhost:9200/_aliases?pretty" -H 'Content-Type: application/json' -d'
    {
        "actions" : [
            { "add":  { "index": "'$index'-reindexed", "alias": "'$index'" } },
            { "remove_index": { "index": "'$index'" } }
        ]
    }'
done
```

### 4.4 Stop version 6.8.0
```bash
docker-compose down
```

### 4.5 Declare elasticsearch environment
```bash
sed -i 's/6.8.0/7.3.1/' elk_env
. ./elk_env
```

### 4.6 Patch elasticsearch.yml
```bash
rm -f /local/elasticsearch/elasticsearch1/config/elasticsearch.yml
cp -p /local/elasticsearch/elasticsearch1/config/elasticsearch.yml.7 elasticsearch.yml

rm -f /local/elasticsearch/elasticsearch2/config/elasticsearch.yml
cp -p /local/elasticsearch/elasticsearch2/config/elasticsearch.yml.7 elasticsearch.yml
```

### 4.7 Start version 7.3.1
```bash
docker-compose up -d
```

---

## 5. Kibana
> Host: App
### 5.1 Patch kibana.yml
```bash
cp -p /local/elasticsearch/kibana/kibana.yml /local/elasticsearch/kibana/kibana.yml.bak

rm -f /local/elasticsearch/kibana/kibana.yml
cp -p kibana.yml.7 /local/elasticsearch/kibana/kibana.yml
```

### 5.2 Start Kibana 7.3.1
```bash
docker-compose -f docker-compose-kib-7_3.yml up -d
```

## 6. Enable cronjob at APP VM
> Host: App
```bash
# DEV
mv /var/spool/cron/crontabs/devadm.disabled /var/spool/cron/crontabs/devadm

# SIT & UAT
mv /var/spool/cron/crontabs/dockadm.disabled /var/spool/cron/crontabs/dockadm
```




