# EVM (BASE) Outpost Quickstart

Jackal Outposts are smart contracts that allow other blockchains to connect to the Jackal data layer.

{% hint style="info" %}
Note: Jackal EVM Outposts are currently available only on BASE mainnet network.
{% endhint %}


{% hint style="info" %}
Jackal Outposts are in an open beta state. Please be aware that this documentation might change.
{% endhint %}

## Quickstart

1. [Deploy bridge contract](https://github.com/JackalLabs/mulberry/blob/main/DEPLOY.md)
   or [find a bridge](https://github.com/JackalLabs/mulberry/blob/main/config/config.go#L43) from EVM chain -> Jackal

2. Using [Wagmi](https://wagmi.sh/) to connect React to the EVM chain is recommended
    ```bash
    npm install wagmi viem@2.x @tanstack/react-query
    ```

3. Configure Wagmi [as recommended](https://wagmi.sh/react/getting-started#use-wagmi) or use
   the [demo config](https://github.com/JackalLabs/evm-demo-ui/blob/main/src/wagmi.ts)

4. Import the hooks [useReadContract](https://wagmi.sh/react/api/hooks/useReadContract)
   and [useWriteContract](https://wagmi.sh/react/api/hooks/useWriteContract)
    ```ts
    import { useReadContract, useWriteContract } from "wagmi";
    
    const { data, error, isPending, writeContract } = useWriteContract();
    ```

5. Locate the [ABI](https://docs.soliditylang.org/en/latest/abi-spec.html) for the bridge contract (
   RootABI [here](https://github.com/JackalLabs/evm-demo-ui/blob/main/src/app/abis.ts))

6. Export the ABI for your storage
   contract ([Vyper instructions](https://docs.vyperlang.org/en/latest/deploying-contracts.html#deploying-a-contract))
    ```bash
    forge inspect [contract name] abi
    ```

7. Check if the user already has an allowance set with `useReadContract`
    ```ts
    const {refetch, data, isFetched} = useReadContract({
        abi: [bridge contract abi],
        address: [bridge contract address],
        functionName: 'getAllowance',
        args: [[storage contract address], [user address]], // keep outer brackets 
        chainId: [network id],
    })
    ```

8. If not, set an allowance for the user using `writeContract`
    ```ts
    await writeContract({
        abi: [bridge contract abi],
        address: [bridge contract address],
        functionName: 'addAllowance',
        args: [[storage contract address]], // keep outer brackets 
        chainId: [network id],
    })
    ```

9. After that, you can call any storage contract function

10. Here we call `upload` from
    the [example contract](https://github.com/JackalLabs/mulberry/blob/main/forge/src/StorageDrawer.sol#L15)
    ```ts
    await writeContract({
        abi: [storage contract abi],
        address: [bridge contract address],
        functionName: 'upload',
        args: [[file content], [file size]], // keep outer brackets 
        value: [user fee to contract],
        chainId: [network id],
    })
    ```

11. Now you can use [typescript](https://gist.github.com/slightlyskepticalpotat/fa40f705eca2e324f03f881485ecd6a0) to
    interact directly with Jackal.
    ```ts
    import { Merkletree } from "@jackallabs/dogwood-tree";

    // Builds dogwood tree from file
    // react ex:
    // const [file, setFile] = useState<File>(new File([""], ""));
    const tree = await Merkletree.grow({
        seed: await file.arrayBuffer(),
        chunkSize: 10240,
        preserve: false,
    });
    const root = tree.getRootAsHex();
    
    // Create a new WebSocket connection
    const socket = new WebSocket("wss://rpc.jackalprotocol.com/websocket");
    
    // Open websocket connection to RPC node
    socket.addEventListener("open", (event) => {
        const subscriptionMessage = JSON.stringify({
            jsonrpc: "2.0",
            method: "subscribe",
            id: "1",
            params: {
                query: `tm.event='Tx' AND post_file.file='${root}'`,
            },
        });
        socket.send(subscriptionMessage);
    });
    
    // Listen for the EVM chain -> Jackal tx
    socket.addEventListener("message", async (event) => {
        const data = JSON.parse(event.data);
        if (Object.keys(data.result).length == 0) {
            return;
        }

        // URL of provider we upload to
        const url = "https://mprov01.jackallabs.io/v2/upload"; 
        // view all of them (https://api.jackalprotocol.com/jackal/canine-chain/storage/active_providers)
    
        // Build our FormData object
        const formData = new FormData();
        formData.append("file", file);
        formData.append("sender", data.result.events["post_file.signer"][0]);
        formData.append("merkle", root);
        formData.append("start", data.result.events["post_file.start"][0]);

        // Upload file to provider
        const request = new Request(url, {
            method: "POST",
            body: formData,
        });

        // Handle the response
        const response = await fetch(request);
        if (!response.ok) {
            throw new Error(`Upload failed with status: ${response.status}`);
        }
    
        const jobResponseData = await response.json();

        const job = jobResponseData["job_id"]

        const joburl = `https://mprov01.jackallabs.io/v2/status/${job}`

        let complete = false
        while (!complete) {
          const response = await fetch(joburl);
          const data = await response.json();
          const progress = data["progress"];
          if (progress == 100) {
            complete = true
            const cid = data["cid"];
            console.log("cid:", cid)
          } else {
            await new Promise(r => setTimeout(r, 250));
          }
        }
    });
    ```