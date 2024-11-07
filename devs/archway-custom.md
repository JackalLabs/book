# Custom Archway Contracts

If you want to use a custom contract (ie. embedding storage into your own contracts) you will need to follow a contract API that lets Jackal.js call the contracts interchangeably.

Here is a trimmed down version of our `outpost factory` contract. This contract has two main functions, `CreateOutpost` will create a new copy of the ICA controller contract. 
And querying the state of this contract lets us figure out which controller contract to call to upload files.

## Example Custom Contract

```rust
#[cfg(not(feature = "library"))]
use cosmwasm_std::entry_point;
use cosmwasm_std::{to_json_binary, Binary, Deps, DepsMut, Env, MessageInfo, Response, StdResult};

use crate::error::ContractError;
use crate::msg::{ExecuteMsg, InstantiateMsg, QueryMsg};
use crate::state::{ContractState, STATE};

#[cfg_attr(not(feature = "library"), entry_point)]
pub fn instantiate(
    deps: DepsMut,
    _env: Env,
    info: MessageInfo,
    msg: InstantiateMsg,
) -> Result<Response, ContractError> {
    STATE.save(
        deps.storage,
        &ContractState::new(msg.storage_outpost_code_id, info.sender.to_string()),
    )?;
    Ok(Response::default())
}

#[cfg_attr(not(feature = "library"), entry_point)]
pub fn execute(
    deps: DepsMut,
    env: Env,
    info: MessageInfo,
    msg: ExecuteMsg,
) -> Result<Response, ContractError> {
    match msg {
        ExecuteMsg::CreateOutpost {
            channel_open_init_options,
        } => execute::create_outpost(deps, env, info, channel_open_init_options),
        ExecuteMsg::MapUserOutpost { outpost_owner} => execute::map_user_outpost(deps, env, info, outpost_owner),
    }
}

#[cfg_attr(not(feature = "library"), entry_point)]
pub fn query(deps: Deps, _env: Env, msg: QueryMsg) -> StdResult<Binary> {
    match msg {
        QueryMsg::GetContractState {} => to_json_binary(&query::state(deps)?),
        QueryMsg::GetUserOutpostAddress { user_address } => to_json_binary(&query::user_outpost_address(deps, user_address)?),
        QueryMsg::GetAllUserOutpostAddresses {  } => to_json_binary(&query::get_all_user_outpost_addresses(deps)?),
    }
}

mod execute {
    use cosmwasm_std::{Addr, BankMsg, Coin, CosmosMsg, Uint128, Event, to_json_binary};
    use storage_outpost::outpost_helpers::StorageOutpostContract;
    use storage_outpost::types::msg::ExecuteMsg as IcaControllerExecuteMsg;
    use storage_outpost::types::state::{CallbackCounter, ChannelState};
    use storage_outpost::{
        outpost_helpers::StorageOutpostCode,
        types::msg::options::ChannelOpenInitOptions,
    };
    use storage_outpost::types::callback::Callback;
    use serde_json_wasm::from_str;

    use crate::state::{self, USER_ADDR_TO_OUTPOST_ADDR, LOCK};

    use super::*;
    pub fn create_outpost(
        deps: DepsMut,
        env: Env,
        info: MessageInfo,
        channel_open_init_options: ChannelOpenInitOptions,
    ) -> Result<Response, ContractError> {
        let state = STATE.load(deps.storage)?;

        let storage_outpost_code_id = StorageOutpostCode::new(state.storage_outpost_code_id);

        if let Some(value) = USER_ADDR_TO_OUTPOST_ADDR.may_load(deps.storage, &info.sender.to_string())? {
            return Err(ContractError::AlreadyCreated(value))
        }

        let _lock = LOCK.save(deps.storage, &info.sender.to_string(), &true);

        let callback = Callback {
            contract: env.contract.address.to_string(),
            msg: None,
            outpost_owner: info.sender.to_string(),
        };

        let instantiate_msg = storage_outpost::types::msg::InstantiateMsg {
            owner: Some(info.sender.to_string()),
            admin: Some(env.contract.address.to_string()),
            channel_open_init_options: Some(channel_open_init_options),
            callback: Some(callback),
        };

        let label
            = format!("storage_outpost-owned by: {}", &info.sender.to_string());

        let cosmos_msg = storage_outpost_code_id.instantiate(
            instantiate_msg,
            label,
            Some(env.contract.address.to_string()),
        )?;

        let mut event = Event::new("FACTORY: create_ica_contract");
        event = event.add_attribute("info.sender", &info.sender.to_string());

        Ok(Response::new().add_message(cosmos_msg).add_event(event))
    }

    pub fn map_user_outpost(
        deps: DepsMut,
        env: Env,
        info: MessageInfo,
        outpost_owner: String,
    ) -> Result<Response, ContractError> {
        let lock = LOCK.may_load(deps.storage, &outpost_owner)?;

        if let Some(true) = lock {
            LOCK.save(deps.storage, &outpost_owner, &false)?;
        } else {
            return Err(ContractError::MissingLock {})
        }

        USER_ADDR_TO_OUTPOST_ADDR.save(deps.storage, &outpost_owner, &info.sender.to_string())?;

        let mut event = Event::new("FACTORY:map_user_outpost");
        event = event.add_attribute("info.sender", &info.sender.to_string());

        Ok(Response::new().add_event(event))
    }

    mod query {
        use crate::state::{USER_ADDR_TO_OUTPOST_ADDR};

        use super::*;

        pub fn state(deps: Deps) -> StdResult<ContractState> {
            STATE.load(deps.storage)
        }

        pub fn user_outpost_address(deps: Deps, user_address: String) -> StdResult<String> {
            USER_ADDR_TO_OUTPOST_ADDR.load(deps.storage, &user_address)
        }

        pub fn get_all_user_outpost_addresses(deps: Deps) -> StdResult<Vec<(String, String)>> {
            let mut all_entries = Vec::new();

            let pairs = USER_ADDR_TO_OUTPOST_ADDR
                .range(deps.storage, None, None, cosmwasm_std::Order::Ascending);

            for pair in pairs {
                let (key, value) = pair?;
                all_entries.push((key.to_string(), value));
            }

            Ok(all_entries)
        }
    }
}
```

Please refer to this GitHub repo for more information: https://github.com/JackalLabs/storage-outpost/blob/master/cross-contract/contracts/outpost-factory/src/contract.rs

## Integrating with Jackal.js

When using a custom contract, you can make Jackal.js use it instead of the default Outpost Factory. It is as easy as adding a `contract` field to the `createWasmStorageHandler` function call.

```typescript
myStorageHandler.createWasmStorageHandler({contract: 'YOUR_CONTRACT_ADDRESS_HERE'})
```