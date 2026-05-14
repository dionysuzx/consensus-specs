# Operations tests

The different kinds of operations ("transactions") are tested individually with
test handlers.

## Test case format

### `meta.yaml`

```yaml
description: string    -- Optional description of test case, purely for debugging purposes.
                          Tests should use the directory name of the test case as identifier, not the description.
bls_setting: int       -- see general test-format spec.
```

### `pre.ssz_snappy`

An SSZ-snappy encoded `BeaconState`, the state before applying the operation.

### `<input-name>.ssz_snappy`

An SSZ-snappy encoded operation object, e.g. a `ProposerSlashing`, or `Deposit`.

### `post.ssz_snappy`

An SSZ-snappy encoded `BeaconState`, the state after applying the operation. No
value if operation processing is aborted.

## Condition

A handler of the `operations` test-runner should process these cases, calling
the corresponding processing implementation. This excludes the other parts of
the block-transition.

Operations:

| *`operation-name`*         | *`operation-object`*         | *`input name`*          | *`processing call`*                                                                 |
| -------------------------- | ---------------------------- | ----------------------- | ----------------------------------------------------------------------------------- |
| `attestation`              | `Attestation`                | `attestation`           | `process_attestation(state, attestation)`                                           |
| `attester_slashing`        | `AttesterSlashing`           | `attester_slashing`     | `process_attester_slashing(state, attester_slashing)`                               |
| `block_header`             | `BeaconBlock`                | **`block`**             | `process_block_header(state, block)`                                                |
| `deposit`                  | `Deposit`                    | `deposit`               | `process_deposit(state, deposit)`                                                   |
| `proposer_slashing`        | `ProposerSlashing`           | `proposer_slashing`     | `process_proposer_slashing(state, proposer_slashing)`                               |
| `voluntary_exit`           | `SignedVoluntaryExit`        | `voluntary_exit`        | `process_voluntary_exit(state, voluntary_exit)`                                     |
| `sync_aggregate`           | `SyncAggregate`              | `sync_aggregate`        | `process_sync_aggregate(state, sync_aggregate)` (new in Altair)                     |
| `execution_payload`        | `BeaconBlockBody`            | **`body`**              | `process_execution_payload(state, body)` (new in Bellatrix, removed in Gloas)       |
| `withdrawals`              | `ExecutionPayload`           | `execution_payload`     | `process_withdrawals(state, execution_payload)` (new in Capella, modified in Gloas) |
| `bls_to_execution_change`  | `SignedBLSToExecutionChange` | `address_change`        | `process_bls_to_execution_change(state, address_change)` (new in Capella)           |
| `deposit_request`          | `DepositRequest`             | `deposit_request`       | `process_deposit_request(state, deposit_request)` (new in Electra)                  |
| `withdrawal_request`       | `WithdrawalRequest`          | `withdrawal_request`    | `process_withdrawal_request(state, withdrawal_request)` (new in Electra)            |
| `consolidation_request`    | `ConsolidationRequest`       | `consolidation_request` | `process_consolidation_request(state, consolidation_request)` (new in Electra)      |
| `execution_payload_bid`    | `BeaconBlock`                | **`block`**             | `process_execution_payload_bid(state, block)` (new in Gloas)                        |
| `parent_execution_payload` | `BeaconBlock`                | **`block`**             | `process_parent_execution_payload(state, block)` (new in Gloas)                     |
| `payload_attestation`      | `PayloadAttestation`         | `payload_attestation`   | `process_payload_attestation(state, payload_attestation)` (new in Gloas)            |

Note that some entries are not strictly operations, but are processed in the
same manner and included here. For example, `block_header` takes a full
`BeaconBlock`.

In Gloas, `BeaconBlockBody` carries a `SignedExecutionPayloadBid` instead of an
`ExecutionPayload`, and carries `parent_execution_requests` for the parent
payload. The current block's execution payload is revealed separately as a
`SignedExecutionPayloadEnvelope`; envelope test cases are documented in
[`fork_choice`](../fork_choice/README.md). Also, `withdrawals` is processed as
`process_withdrawals(state)`, without an `execution_payload` input.

In forks where it is present, the `execution_payload` handler requires execution
payload validation by the `ExecutionEngine`. During testing, that validation is
mocked with an `execution.yml` file containing an `execution_valid` boolean with
the verification result.

The resulting state should match the expected `post` state, or if the `post`
state is left blank, the handler should reject the input operation as invalid.
