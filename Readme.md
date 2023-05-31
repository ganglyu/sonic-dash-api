# sonic-dash-api

This repository hosts the DASH API definition for the SONiC project. The schema of DASH APP DB is at [DASH APP DB](https://github.com/sonic-net/DASH/blob/main/documentation/general/dash-sonic-hld.md#32-dash-app-db) and all entries of DASH APP DB will be encoded as protobuf.

## Protobuf Convention

1. File name use underscore case, E.G. `acl_rule.proto`
2. All file except common utility should include and only include one message of entry and one message of its key.
3. Message name of entry use camel case. E.G. `AclRule`.
4. Message name of entry key use the entry name with a fixed postfix `Key`. E.G. `AclRuleKey`.
5. If the value of entry is a list, the item use the entry name with a fixed postfix `Item`. E.G. `RouteTypeItem`.
6. Member variable use underscore case. E.G. `src_addr`.
7. For enumerations type, the enum name use camel case, E.G. `IpVersion`
8. The field of enum use full capital with underscore case, and the enum name use as the prefix for each field. E.G. 
`IP_VERSION_IPV4`

## Redis DB

1. Table name will be full capital with underscore. And the prefix `DASH` and postfix `Table` will be added to entry name. E.G. `DASH_VNET_MAPPING_TABLE`
2. The key message is sequentially joint as the Redis key with colon separator. E.G. `AclRuleKey{group_id=group1, rule_num=3}`, the key of Redis entry will be `DASH_ACL_RULE_TABLE:group1:3`
3. The value is the entry message with the bytes array of protobuf

## GNMI

```
update { 
    path { 
        origin: "sonic_db" 
        elem {name: "DPU0"} elem {name: "APPL_DB"} elem {name: "DASH_ACL_RULE_TABLE"} elem {name: "group1:3"} 
    } 
    val { 
        proto_bytes: “\n\x010\x12$b6d54023-5d24-47de-ae94-8afe693dd1fc…” 
    } 
} 
```
1. GNMI path will have 4 elements, please refer to the above example
2. The first element is device name, and it should be one of the following values: `localhost`, `DPU0`, `DPU1`, ..., `DPUx`
3. The second element is SONiC database name, and it should be `APPL_DB` for DASH API
4. The third element is table name, please refer to [DASH APP DB](https://github.com/sonic-net/DASH/blob/main/documentation/general/dash-sonic-hld.md#32-dash-app-db)
5. The fourth element is table key. For example, in the `DASH_ACL_RULE_TABLE`, the table key consists of two fields: `group_id` and `rule_num`. These fields are separated by a colon `:` in the APPL_DB. Therefore, the fourth element would be `group1:3`
6. GNMI value is the bytes array of protobuf
