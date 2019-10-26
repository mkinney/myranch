## ETH2 Testnet from Prysm Labs

See https://prylabs.net/participate

- Deploy persistent volume, persistent volume claim, and job to create the raw transaction data to activate validator client
kubectl apply -f bottometh2_pv.yml -f bottometh2_pvc.yml -f bottometh2_j.yml

- Wait until job (bottometh2init) is complete (not 100% sure need to do this, just following their instructions)

- Try to hit the RPC endpoint: (not working)

    ```
    curl -H "Content-Type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"web3_clientVersion","params":[],"id":67}' 'http://bottom.mikekinney.org:7000'
    ```

- Deploy the beacon chain

    kubectl apply -f bottometh2_d.yml

- Wait for chain to sync (can take a couple of hours)

- Need to manually send some test eth (need the output from the job above) for validator

- Deploy validator

    kubectl apply -f bottometh2_d2.yml

- Remove all eth2:

    delete -f bottometh2_d.yml -f bottometh2_d2.yml -f bottometh2_pv.yml -f bottometh2_j.yml -f bottometh2_pvc.yml
