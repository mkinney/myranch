Spin up ethereum nodes (note: added node affinity so we keep the persisted volume on same host)

    ```
    rancher kubectl apply -f eth1/topeth1_d.yml -f eth1/topeth1_pvc.yml -f eth1/topeth1_pv.yml

    rancher kubectl apply -f eth1/bottometh1_d.yml -f eth1/bottometh1_pvc.yml -f eth1/bottometh1_pv.yml
    ```

Test ethereum rpc is working (see https://github.com/ethereum/wiki/wiki/JSON-RPC#web3_clientversion for more commands)

    ```
    curl -H "Content-Type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"web3_clientVersion","params":[],"id":67}' 'http://192.168.0.202:8545'

    {"jsonrpc":"2.0","id":67,"result":"Geth/v1.9.7-unstable-db79143a-20191024/linux-amd64/go1.13.3"}
    ```
