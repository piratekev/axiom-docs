---
description: Details about pricing, limits, and gas when using Axiom
sidebar_position: 2
sidebar_label: Gas, Pricing, and Limits
---

# Gas, Pricing, and Limits

Axiom charges fees and verifies proofs on-chain via the `AxiomV2Query` contract, which manages balances, tracks query state, and escrows/unescrows ETH from your balance as queries are requested and fulfilled.

## Fulfillment Gas and Pricing

The fulfillment cost of an Axiom query includes:

* gas for on-chain verification of the final ZK proof generated by Axiom
* gas to trigger the final smart contract callback
* a fee charged by Axiom on a per-query basis 

Axiom passes through gas costs to the user and collects an additional fee for each query.  The `AxiomV2Query` contract escrows funds for each query from your balance to ensure that you can cover the payment. This amount is calculated as

```
maxQueryPri = maxFeePerGas * (callbackGasLimit + proofVerificationGas) + axiomQueryFee;
```

The parameters set by Axiom are:

- `proofVerificationGas`: The gas cost of ZK proof verification, currently set to `420_000` gas
- `axiomQueryFee`: Fee charged by Axiom, fixed to `0.003 ether`

The parameters set by the user for each query are:

- `maxFeePerGas`: The max fee per gas that the prover should use in the fulfill query transaction.
- `callbackGasLimit`: The gas limit allocated for use in the callback.

If either of these parameters is too low, the fulfill query transaction cannot be sent successfully. Prior to query fulfillment, the user can update the parameters `maxFeePerGas` and `callbackGasLimit` by calling the `increaseQueryGas` function.

The `AxiomV2Query` contract will escrow `maxQueryPri` from the user's balance for a query. This is the maximum cost of fulfilling the query. After the prover has fulfilled the query, they can refund any
portion of the `maxQueryPri` not used in gas or the `axiomQueryFee` back to the `refundee` specified in the query. The refund is sent to the user's balance.

## Query Limits

*We can support much larger queries on a partnership basis.  If you are interested in larger queries, fill out the Axiom [partner form](https://airtable.com/shrdqI16f6EZBNkMA) to explore an integration.*

For the Axiom V2 mainnet release, an Axiom query has the following limits:

- The `axiomResults: bytes32[]` sent in the callback can have length at most **`128`**.
- There can be at most **`128`** total subqueries in the query.

These are the only limits when there are no subqueries of the following types:

- Transaction subquery for a transaction with more than `8192` bytes of input data or more than `4096` bytes in the RLP encoded access list.
- Receipt subquery for a transaction receipt with either more than `80` logs or which contains a log with data field exceeding `1024` bytes.

For these larger transaction and receipt subqueries, we have more fine-grained limits. These are detailed in the [Transaction Subquery](/sdk/typescript-sdk/axiom-circuit/axiom-subqueries/transaction-subquery) and [Receipt Subquery](/sdk/typescript-sdk/axiom-circuit/axiom-subqueries/receipt-subquery) sections of the Axiom Typescript SDK documentation.

## Deposit and Withdraw

You can deposit ETH into the `AxiomV2Query` contract by calling the `deposit` function or by transfering ETH together with your on-chain query. You can withdraw from your unescrowed balance by calling the `withdraw` function.

## Refunds

If a query is not fulfilled by `deadlineBlockNumber`, the `refundee` specified in the query can retrieve their fees paid using the `refundQuery` function.

The `deadlineBlockNumber` is set to `queryDeadlineInterval` blocks after query submission. Currently the value of `queryDeadlineInterval` is `7200` blocks (approximately 1 day).