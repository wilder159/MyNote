```json
{
    "iotId":"4z819VQHk6VSLmmBJfrf00107e****",
    "requestId":"2",
    "productKey":"al12345****",
    "deviceName":"deviceName1234",
    "gmtCreate":1510799670074,
    "deviceType":"Ammeter",
    "items":{
        "Power":{
            "value":"on",
            "time":1510799670074
        },
        "Position":{
            "time":1510292697470,
            "value":{
                "latitude":39.9,
                "longitude":116.38
            }
        }
    },
    "checkFailedData":{
        "attribute_8":{
            "time": 1510292697470,
            "value": 715665571,
            "code":6304,
            "message":"tsl parse: params not exist -> attribute_8"
        }
    }
}
```

 697个字符   
 21个数据

```c
/* The cJSON structure: */
typedef struct cJSON
{
    /* next/prev allow you to walk array/object chains. Alternatively, use GetArraySize/GetArrayItem/GetObjectItem */
    struct cJSON *next;  4
    struct cJSON *prev;  4
    /* An array or object item will have a child pointer pointing to a chain of the items in the array/object. */
    struct cJSON *child; 4

    /* The type of the item, as above. */
    int type;            4

    /* The item's string, if type==cJSON_String  and type == cJSON_Raw */
    char *valuestring;   4
    /* writing to valueint is DEPRECATED, use cJSON_SetNumberValue instead */
    int valueint;        4
    /* The item's number, if type==cJSON_Number */
    double valuedouble;  8

    /* The item's name string, if this item is the child of, or is in the list of subitems of an object. */
    char *string;        4
} cJSON;

```

占用空间  9*4 = 36个字节
36 * 20 = 720 字节 

C