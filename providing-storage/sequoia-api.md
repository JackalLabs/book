# Sequoia API

## GET
### Index `/`
#### Description
Simple details about the storage provider.
#### Parameters
| name | type |
|------|------|
| None | n/a  |
#### Response
| name      | type   | desc.                                                        |
|-----------|--------|--------------------------------------------------------------|
| `status`  | string | Will always say "online" due to the response being available |
| `address` | string | The address of the storage provider                          |

### Version `/version`
#### Description
Lists the version of the software the storage provider is running.
#### Parameters
| name | type |
|------|------|
| None | n/a  |
#### Response
| name        | type     | desc.                                                |
|-------------|----------|------------------------------------------------------|
| `version`   | string   | The git tag version (ex. v1.0.2)                     |
| `build`     | string   | The git commit                                       |
| `chain-id`  | string   | The tendermint chain-id the provider is connected to |

### Download File `/download`
#### Description
Downloads a file from the Jackal Protocol
#### Parameters
| name     | type       |
|----------|------------|
| `merkle` | hex string |
#### Response
| name       | type | desc.              |
|------------|------|--------------------|
| `file`     | file | The requested file |


### List Files `/list`
#### Description
Lists every file stored on this provider.
#### Parameters
| name | type |
|------|------|
| None | n/a  |
#### Response
| name       | type         | desc.                                                     |
|------------|--------------|-----------------------------------------------------------|
| `files`    | string array | Every merkle hash stored on this provider, as hex strings |


### List IPFS Peers `/ipfs/peers`
#### Description
Lists every connected IPFS peer ID.
#### Parameters
| name | type |
|------|------|
| None | n/a  |
#### Response
| name    | type         | desc.                        |
|---------|--------------|------------------------------|
| `peers` | string array | Every IPFS peer ID in base64 |

### List IPFS Peers `/ipfs/hosts`
#### Description
Lists the hosts this storage provider is accessible from.
#### Parameters
| name | type |
|------|------|
| None | n/a  |
#### Response
| name    | type         | desc.                                                   |
|---------|--------------|---------------------------------------------------------|
| `hosts` | string array | List of multiaddrs that this provider can be reached at |


### List IPFS Files `/ipfs/cids`
#### Description
Lists every file stored on this device as an IFPS CID.
#### Parameters
| name | type |
|------|------|
| None | n/a  |
#### Response
| name   | type         | desc.                                     |
|--------|--------------|-------------------------------------------|
| `cids` | string array | List of IPFS CIDs stored on this provider |

### Map IPFS & Jackal File IDs `/ipfs/cid_map`
#### Description
Returns a map of all files' Merkles as hex strings paired to IPFS CIDs.
#### Parameters
| name | type |
|------|------|
| None | n/a  |
#### Response
| name      | type                 | desc.                                            |
|-----------|----------------------|--------------------------------------------------|
| `cid_map` | map <string, string> | Mapping file merkles as hex strings to IPFS CIDs |


### Dump Database State `/dump`
#### Description
Dumps the entire database state to JSON.
#### Parameters
| name | type |
|------|------|
| None | n/a  |
#### Response
| name   | type        | desc.               |
|--------|-------------|---------------------|
| `dump` | JSON Object | Entire DB as object |

### Prometheus Metrics `/metrics`
#### Description
Prometheus formatted metrics.
#### Parameters
| name | type |
|------|------|
| None | n/a  |
#### Response
| name      | type            | desc.                   |
|-----------|-----------------|-------------------------|
| `metrics` | Prometheus File | Prometheus metrics dump |

## POST

### Upload Files `/upload`
#### Description
Upload a file to Jackal.
#### Parameters (multi-part form format)
| name     | type                              |
|----------|-----------------------------------|
| `sender` | Jackal Address string             |
| `merkle` | Merkle Root of file in hex string |
| `start`  | Start block of file deal          |
#### Response
| name     | type                  | desc.                    |
|----------|-----------------------|--------------------------|
| `merkle` | Hex String            | Merkle of file           |
| `Owner`  | Jackal Address String | Owner of file deal       |
| `Start`  | Integer               | Start block of file deal |
| `CID`    | IPFS CID String       | IPFS CID of file deal    |

### Create IPFS Folder `/ipfs/make_folder`
#### Description
Create an IPFS folder mapping.
#### Parameters (plaintext)
| name     | type                              |
|----------|-----------------------------------|
| `n/a`    | Comma delimited list of IPFS CIDs |
#### Response
| name     | type                  | desc.              |
|----------|-----------------------|--------------------|
| `CID`    | IPFS CID String       | IPFS CID of folder |
