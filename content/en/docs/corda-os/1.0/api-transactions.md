---
aliases:
- /releases/release-V1.0/api-transactions.html
date: '2020-01-08T09:59:25Z'
menu:
  corda-os-1-0:
    parent: corda-os-1-0-api
tags:
- api
- transactions
title: 'API: Transactions'
---




# API: Transactions

{{< note >}}
Before reading this page, you should be familiar with the key concepts of [Transactions](key-concepts-transactions.md).

{{< /note >}}


## Transaction lifecycle

Between its creation and its final inclusion on the ledger, a transaction will generally occupy one of three states:


* `TransactionBuilder`. A transaction’s initial state. This is the only state during which the transaction is
mutable, so we must add all the required components before moving on.
* `SignedTransaction`. The transaction now has one or more digital signatures, making it immutable. This is the
transaction type that is passed around to collect additional signatures and that is recorded on the ledger.
* `LedgerTransaction`. The transaction has been “resolved” - for example, its inputs have been converted from
references to actual states - allowing the transaction to be fully inspected.

We can visualise the transitions between the three stages as follows:

![transaction flow](/en/images/transaction-flow.png "transaction flow")

## Transaction components

A transaction consists of six types of components:


* 1+ states:
    * 0+ input states
    * 0+ output states


* 1+ commands
* 0+ attachments
* 0 or 1 time-window
    * A transaction with a time-window must also have a notary



Each component corresponds to a specific class in the Corda API. The following section describes each component class,
and how it is created.


### Input states

An input state is added to a transaction as a `StateAndRef`, which combines:


* The `ContractState` itself
* A `StateRef` identifying this `ContractState` as the output of a specific transaction

{{< tabs name="tabs-1" >}}
{{% tab name="kotlin" %}}
```kotlin
ourStateAndRef: StateAndRef<DummyState> = serviceHub.toStateAndRef<DummyState>(ourStateRef)

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L242-L242' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 21' end='DOCEND 21' */}}
{{% tab name="java" %}}
```java
StateAndRef ourStateAndRef = getServiceHub().toStateAndRef(ourStateRef);

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L253-L253' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 21' end='DOCEND 21' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

A `StateRef` uniquely identifies an input state, allowing the notary to mark it as historic. It is made up of:


* The hash of the transaction that generated the state
* The state’s index in the outputs of that transaction

{{< tabs name="tabs-2" >}}
{{% tab name="kotlin" %}}
```kotlin
ourStateRef: StateRef = StateRef(SecureHash.sha256("DummyTransactionHash"), 0)

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L238-L238' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 20' end='DOCEND 20' */}}
{{% tab name="java" %}}
```java
StateRef ourStateRef = new StateRef(SecureHash.sha256("DummyTransactionHash"), 0);

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L249-L249' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 20' end='DOCEND 20' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

The `StateRef` links an input state back to the transaction that created it. This means that transactions form
“chains” linking each input back to an original issuance transaction. This allows nodes verifying the transaction
to “walk the chain” and verify that each input was generated through a valid sequence of transactions.


### Output states

Since a transaction’s output states do not exist until the transaction is committed, they cannot be referenced as the
outputs of previous transactions. Instead, we create the desired output states as `ContractState` instances, and
add them to the transaction directly:

{{< tabs name="tabs-3" >}}
{{% tab name="kotlin" %}}
```kotlin
ourOutputState: DummyState = DummyState()

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L252-L252' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 22' end='DOCEND 22' */}}
{{% tab name="java" %}}
```java
DummyState ourOutputState = new DummyState();

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L263-L263' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 22' end='DOCEND 22' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

In cases where an output state represents an update of an input state, we may want to create the output state by basing
it on the input state:

{{< tabs name="tabs-4" >}}
{{% tab name="kotlin" %}}
```kotlin
ourOtherOutputState: DummyState = ourOutputState.copy(magicNumber = 77)

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L256-L256' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 23' end='DOCEND 23' */}}
{{% tab name="java" %}}
```java
DummyState ourOtherOutputState = ourOutputState.copy(77);

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L267-L267' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 23' end='DOCEND 23' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

Before our output state can be added to a transaction, we need to associate it with a contract. We can do this by
wrapping the output state in a `StateAndContract`, which combines:


* The `ContractState` representing the output states
* A `String` identifying the contract governing the state

{{< tabs name="tabs-5" >}}
{{% tab name="kotlin" %}}
```kotlin
ourOutput: StateAndContract = StateAndContract(ourOutputState, DUMMY_PROGRAM_ID)

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L261-L261' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 47' end='DOCEND 47' */}}
{{% tab name="java" %}}
```java
StateAndContract ourOutput = new StateAndContract(ourOutputState, DUMMY_PROGRAM_ID);

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L272-L272' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 47' end='DOCEND 47' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}


### Commands

A command is added to the transaction as a `Command`, which combines:


* A `CommandData` instance indicating the command’s type
* A `List<PublicKey>` representing the command’s required signers

{{< tabs name="tabs-6" >}}
{{% tab name="kotlin" %}}
```kotlin
commandData: DummyContract.Commands.Create = DummyContract.Commands.Create()
ourPubKey: PublicKey = serviceHub.myInfo.legalIdentitiesAndCerts.first().owningKey
counterpartyPubKey: PublicKey = counterparty.owningKey
requiredSigners: List<PublicKey> = listOf(ourPubKey, counterpartyPubKey)
ourCommand: Command<DummyContract.Commands.Create> = Command(commandData, requiredSigners)

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L268-L272' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 24' end='DOCEND 24' */}}
{{% tab name="java" %}}
```java
DummyContract.Commands.Create commandData = new DummyContract.Commands.Create();
PublicKey ourPubKey = getServiceHub().getMyInfo().getLegalIdentitiesAndCerts().get(0).getOwningKey();
PublicKey counterpartyPubKey = counterparty.getOwningKey();
List<PublicKey> requiredSigners = ImmutableList.of(ourPubKey, counterpartyPubKey);
Command<DummyContract.Commands.Create> ourCommand = new Command<>(commandData, requiredSigners);

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L279-L283' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 24' end='DOCEND 24' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}


### Attachments

Attachments are identified by their hash:

{{< tabs name="tabs-7" >}}
{{% tab name="kotlin" %}}
```kotlin
ourAttachment: SecureHash = SecureHash.sha256("DummyAttachment")

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L288-L288' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 25' end='DOCEND 25' */}}
{{% tab name="java" %}}
```java
SecureHash ourAttachment = SecureHash.sha256("DummyAttachment");

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L299-L299' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 25' end='DOCEND 25' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

The attachment with the corresponding hash must have been uploaded ahead of time via the node’s RPC interface.


### Time-windows

Time windows represent the period during which the transaction must be notarised. They can have a start and an end
time, or be open at either end:

{{< tabs name="tabs-8" >}}
{{% tab name="kotlin" %}}
```kotlin
ourTimeWindow: TimeWindow = TimeWindow.between(Instant.MIN, Instant.MAX)
ourAfter: TimeWindow = TimeWindow.fromOnly(Instant.MIN)
ourBefore: TimeWindow = TimeWindow.untilOnly(Instant.MAX)

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L293-L295' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 26' end='DOCEND 26' */}}
{{% tab name="java" %}}
```java
TimeWindow ourTimeWindow = TimeWindow.between(Instant.MIN, Instant.MAX);
TimeWindow ourAfter = TimeWindow.fromOnly(Instant.MIN);
TimeWindow ourBefore = TimeWindow.untilOnly(Instant.MAX);

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L306-L308' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 26' end='DOCEND 26' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

We can also define a time window as an `Instant` plus/minus a time tolerance (e.g. 30 seconds):

{{< tabs name="tabs-9" >}}
{{% tab name="kotlin" %}}
```kotlin
ourTimeWindow2: TimeWindow = TimeWindow.withTolerance(serviceHub.clock.instant(), 30.seconds)

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L301-L301' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 42' end='DOCEND 42' */}}
{{% tab name="java" %}}
```java
TimeWindow ourTimeWindow2 = TimeWindow.withTolerance(getServiceHub().getClock().instant(), Duration.ofSeconds(30));

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L314-L314' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 42' end='DOCEND 42' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

Or as a start-time plus a duration:

{{< tabs name="tabs-10" >}}
{{% tab name="kotlin" %}}
```kotlin
ourTimeWindow3: TimeWindow = TimeWindow.fromStartAndDuration(serviceHub.clock.instant(), 30.seconds)

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L305-L305' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 43' end='DOCEND 43' */}}
{{% tab name="java" %}}
```java
TimeWindow ourTimeWindow3 = TimeWindow.fromStartAndDuration(getServiceHub().getClock().instant(), Duration.ofSeconds(30));

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L318-L318' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 43' end='DOCEND 43' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}


## TransactionBuilder


### Creating a builder

The first step when creating a transaction proposal is to instantiate a `TransactionBuilder`.

If the transaction has input states or a time-window, we need to instantiate the builder with a reference to the notary
that will notarise the inputs and verify the time-window:

{{< tabs name="tabs-11" >}}
{{% tab name="kotlin" %}}
```kotlin
txBuilder: TransactionBuilder = TransactionBuilder(specificNotary)

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L316-L316' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 19' end='DOCEND 19' */}}
{{% tab name="java" %}}
```java
TransactionBuilder txBuilder = new TransactionBuilder(specificNotary);

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L329-L329' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 19' end='DOCEND 19' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

We discuss the selection of a notary in [API: Flows](api-flows.md).

If the transaction does not have any input states or a time-window, it does not require a notary, and can be
instantiated without one:

{{< tabs name="tabs-12" >}}
{{% tab name="kotlin" %}}
```kotlin
txBuilderNoNotary: TransactionBuilder = TransactionBuilder()

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L321-L321' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 46' end='DOCEND 46' */}}
{{% tab name="java" %}}
```java
TransactionBuilder txBuilderNoNotary = new TransactionBuilder();

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L334-L334' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 46' end='DOCEND 46' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}


### Adding items

The next step is to build up the transaction proposal by adding the desired components.

We can add components to the builder using the `TransactionBuilder.withItems` method:

{{< tabs name="tabs-13" >}}
{{% tab name="kotlin" %}}
```kotlin
    /** A more convenient way to add items to this transaction that calls the add* methods for you based on type */
    fun withItems(vararg items: Any): TransactionBuilder {
        for (t in items) {
            when (t) {
                is StateAndRef<*> -> addInputState(t)
                is SecureHash -> addAttachment(t)
                is TransactionState<*> -> addOutputState(t)
                is StateAndContract -> addOutputState(t.state, t.contract)
                is ContractState -> throw UnsupportedOperationException("Removed as of V1: please use a StateAndContract instead")
                is Command<*> -> addCommand(t)
                is CommandData -> throw IllegalArgumentException("You passed an instance of CommandData, but that lacks the pubkey. You need to wrap it in a Command object first.")
                is TimeWindow -> setTimeWindow(t)
                is PrivacySalt -> setPrivacySalt(t)
                else -> throw IllegalArgumentException("Wrong argument type: ${t.javaClass}")
            }
        }
        return this
    }

```
{{% /tab %}}
{{/* github src='core/src/main/kotlin/net/corda/core/transactions/TransactionBuilder.kt' url='https://github.com/corda/corda/blob/1.0/core/src/main/kotlin/net/corda/core/transactions/TransactionBuilder.kt#L58-L75' raw='https://raw.githubusercontent.com/corda/corda/1.0/core/src/main/kotlin/net/corda/core/transactions/TransactionBuilder.kt' start='DOCSTART 1' end='DOCEND 1' */}}

[TransactionBuilder.kt](https://github.com/corda/corda/blob/release/os/1.0/core/src/main/kotlin/net/corda/core/transactions/TransactionBuilder.kt) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

`withItems` takes a `vararg` of objects and adds them to the builder based on their type:


* `StateAndRef` objects are added as input states
* `TransactionState` and `StateAndContract` objects are added as output states
    * Both `TransactionState` and `StateAndContract` are wrappers around a `ContractState` output that link the
output to a specific contract


* `Command` objects are added as commands
* `SecureHash` objects are added as attachments
* A `TimeWindow` object replaces the transaction’s existing `TimeWindow`, if any

Passing in objects of any other type will cause an `IllegalArgumentException` to be thrown.

Here’s an example usage of `TransactionBuilder.withItems`:

{{< tabs name="tabs-14" >}}
{{% tab name="kotlin" %}}
```kotlin
ilder.withItems(
    // Inputs, as ``StateAndRef``s that reference the outputs of previous transactions
    ourStateAndRef,
    // Outputs, as ``StateAndContract``s
    ourOutput,
    // Commands, as ``Command``s
    ourCommand,
    // Attachments, as ``SecureHash``es
    ourAttachment,
    // A time-window, as ``TimeWindow``
    ourTimeWindow


```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L326-L337' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 27' end='DOCEND 27' */}}
{{% tab name="java" %}}
```java
txBuilder.withItems(
        // Inputs, as ``StateAndRef``s that reference to the outputs of previous transactions
        ourStateAndRef,
        // Outputs, as ``StateAndContract``s
        ourOutput,
        // Commands, as ``Command``s
        ourCommand,
        // Attachments, as ``SecureHash``es
        ourAttachment,
        // A time-window, as ``TimeWindow``
        ourTimeWindow
);

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L339-L350' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 27' end='DOCEND 27' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

There are also individual methods for adding components.

Here are the methods for adding inputs and attachments:

{{< tabs name="tabs-15" >}}
{{% tab name="kotlin" %}}
```kotlin
ilder.addInputState(ourStateAndRef)
ilder.addAttachment(ourAttachment)

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L344-L345' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 28' end='DOCEND 28' */}}
{{% tab name="java" %}}
```java
txBuilder.addInputState(ourStateAndRef);
txBuilder.addAttachment(ourAttachment);

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L357-L358' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 28' end='DOCEND 28' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

An output state can be added as a `ContractState`, contract class name and notary:

{{< tabs name="tabs-16" >}}
{{% tab name="kotlin" %}}
```kotlin
ilder.addOutputState(ourOutputState, DUMMY_PROGRAM_ID, specificNotary)

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L350-L350' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 49' end='DOCEND 49' */}}
{{% tab name="java" %}}
```java
txBuilder.addOutputState(ourOutputState, DUMMY_PROGRAM_ID, specificNotary);

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L363-L363' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 49' end='DOCEND 49' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

We can also leave the notary field blank, in which case the transaction’s default notary is used:

{{< tabs name="tabs-17" >}}
{{% tab name="kotlin" %}}
```kotlin
ilder.addOutputState(ourOutputState, DUMMY_PROGRAM_ID)

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L355-L355' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 50' end='DOCEND 50' */}}
{{% tab name="java" %}}
```java
txBuilder.addOutputState(ourOutputState, DUMMY_PROGRAM_ID);

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L368-L368' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 50' end='DOCEND 50' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

Or we can add the output state as a `TransactionState`, which already specifies the output’s contract and notary:

{{< tabs name="tabs-18" >}}
{{% tab name="kotlin" %}}
```kotlin
txState: TransactionState<DummyState> = TransactionState(ourOutputState, DUMMY_PROGRAM_ID, specificNotary)

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L360-L360' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 51' end='DOCEND 51' */}}
{{% tab name="java" %}}
```java
TransactionState txState = new TransactionState(ourOutputState, DUMMY_PROGRAM_ID, specificNotary);

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L373-L373' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 51' end='DOCEND 51' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

Commands can be added as a `Command`:

{{< tabs name="tabs-19" >}}
{{% tab name="kotlin" %}}
```kotlin
ilder.addCommand(ourCommand)

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L365-L365' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 52' end='DOCEND 52' */}}
{{% tab name="java" %}}
```java
txBuilder.addCommand(ourCommand);

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L378-L378' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 52' end='DOCEND 52' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

Or as `CommandData` and a `vararg PublicKey`:

{{< tabs name="tabs-20" >}}
{{% tab name="kotlin" %}}
```kotlin
ilder.addCommand(commandData, ourPubKey, counterpartyPubKey)

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L369-L369' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 53' end='DOCEND 53' */}}
{{% tab name="java" %}}
```java
txBuilder.addCommand(commandData, ourPubKey, counterpartyPubKey);

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L382-L382' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 53' end='DOCEND 53' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

For the time-window, we can set a time-window directly:

{{< tabs name="tabs-21" >}}
{{% tab name="kotlin" %}}
```kotlin
ilder.setTimeWindow(ourTimeWindow)

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L374-L374' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 44' end='DOCEND 44' */}}
{{% tab name="java" %}}
```java
txBuilder.setTimeWindow(ourTimeWindow);

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L387-L387' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 44' end='DOCEND 44' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

Or define the time-window as a time plus a duration (e.g. 45 seconds):

{{< tabs name="tabs-22" >}}
{{% tab name="kotlin" %}}
```kotlin
ilder.setTimeWindow(serviceHub.clock.instant(), 45.seconds)

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L378-L378' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 45' end='DOCEND 45' */}}
{{% tab name="java" %}}
```java
txBuilder.setTimeWindow(getServiceHub().getClock().instant(), Duration.ofSeconds(45));

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L391-L391' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 45' end='DOCEND 45' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}


### Signing the builder

Once the builder is ready, we finalize it by signing it and converting it into a `SignedTransaction`.

We can either sign with our legal identity key:

{{< tabs name="tabs-23" >}}
{{% tab name="kotlin" %}}
```kotlin
onceSignedTx: SignedTransaction = serviceHub.signInitialTransaction(txBuilder)

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L389-L389' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 29' end='DOCEND 29' */}}
{{% tab name="java" %}}
```java
SignedTransaction onceSignedTx = getServiceHub().signInitialTransaction(txBuilder);

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L402-L402' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 29' end='DOCEND 29' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

Or we can also choose to use another one of our public keys:

{{< tabs name="tabs-24" >}}
{{% tab name="kotlin" %}}
```kotlin
otherKey: PublicKey = serviceHub.keyManagementService.freshKey()
onceSignedTx2: SignedTransaction = serviceHub.signInitialTransaction(txBuilder, otherKey)

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L393-L394' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 30' end='DOCEND 30' */}}
{{% tab name="java" %}}
```java
PublicKey otherKey = getServiceHub().getKeyManagementService().freshKey();
SignedTransaction onceSignedTx2 = getServiceHub().signInitialTransaction(txBuilder, otherKey);

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L406-L407' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 30' end='DOCEND 30' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

Either way, the outcome of this process is to create an immutable `SignedTransaction` with our signature over it.


## SignedTransaction

A `SignedTransaction` is a combination of:


* An immutable transaction
* A list of signatures over that transaction

{{< tabs name="tabs-25" >}}
{{% tab name="kotlin" %}}
```kotlin
@CordaSerializable
data class SignedTransaction(val txBits: SerializedBytes<CoreTransaction>,
                             override val sigs: List<TransactionSignature>
) : TransactionWithSignatures {

```
{{% /tab %}}
{{/* github src='core/src/main/kotlin/net/corda/core/transactions/SignedTransaction.kt' url='https://github.com/corda/corda/blob/1.0/core/src/main/kotlin/net/corda/core/transactions/SignedTransaction.kt#L33-L36' raw='https://raw.githubusercontent.com/corda/corda/1.0/core/src/main/kotlin/net/corda/core/transactions/SignedTransaction.kt' start='DOCSTART 1' end='DOCEND 1' */}}

[SignedTransaction.kt](https://github.com/corda/corda/blob/release/os/1.0/core/src/main/kotlin/net/corda/core/transactions/SignedTransaction.kt) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

Before adding our signature to the transaction, we’ll want to verify both the transaction’s contents and the
transaction’s signatures.


### Verifying the transaction’s contents

If a transaction has inputs, we need to retrieve all the states in the transaction’s dependency chain before we can
verify the transaction’s contents. This is because the transaction is only valid if its dependency chain is also valid.
We do this by requesting any states in the chain that our node doesn’t currently have in its local storage from the
proposer(s) of the transaction. This process is handled by a built-in flow called `ReceiveTransactionFlow`.
See [API: Flows](api-flows.md) for more details.

We can now verify the transaction’s contents to ensure that it satisfies the contracts of all the transaction’s input
and output states:

{{< tabs name="tabs-26" >}}
{{% tab name="kotlin" %}}
```kotlin
eSignedTx.verify(serviceHub)

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L469-L469' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 33' end='DOCEND 33' */}}
{{% tab name="java" %}}
```java
    twiceSignedTx.verify(getServiceHub());

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L480-L480' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 33' end='DOCEND 33' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

Checking that the transaction meets the contract constraints is only part of verifying the transaction’s contents. We
will usually also want to perform our own additional validation of the transaction contents before signing, to ensure
that the transaction proposal represents an agreement we wish to enter into.

However, the `SignedTransaction` holds its inputs as `StateRef` instances, and its attachments as `SecureHash`
instances, which do not provide enough information to properly validate the transaction’s contents. We first need to
resolve the `StateRef` and `SecureHash` instances into actual `ContractState` and `Attachment` instances, which
we can then inspect.

We achieve this by using the `ServiceHub` to convert the `SignedTransaction` into a `LedgerTransaction`:

{{< tabs name="tabs-27" >}}
{{% tab name="kotlin" %}}
```kotlin
ledgerTx: LedgerTransaction = twiceSignedTx.toLedgerTransaction(serviceHub)

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L483-L483' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 32' end='DOCEND 32' */}}
{{% tab name="java" %}}
```java
    LedgerTransaction ledgerTx = twiceSignedTx.toLedgerTransaction(getServiceHub());

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L494-L494' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 32' end='DOCEND 32' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

We can now perform our additional verification. Here’s a simple example:

{{< tabs name="tabs-28" >}}
{{% tab name="kotlin" %}}
```kotlin
outputState: DummyState = ledgerTx.outputsOfType<DummyState>().single()
outputState.magicNumber == 777) {
// ``FlowException`` is a special exception type. It will be
// propagated back to any counterparty flows waiting for a
// message from this flow, notifying them that the flow has
// failed.
throw FlowException("We expected a magic number of 777.")


```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L488-L495' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 34' end='DOCEND 34' */}}
{{% tab name="java" %}}
```java
    DummyState outputState = ledgerTx.outputsOfType(DummyState.class).get(0);
    if (outputState.getMagicNumber() != 777) {
        // ``FlowException`` is a special exception type. It will be
        // propagated back to any counterparty flows waiting for a
        // message from this flow, notifying them that the flow has
        // failed.
        throw new FlowException("We expected a magic number of 777.");
    }

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L499-L506' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 34' end='DOCEND 34' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}


### Verifying the transaction’s signatures

Aside from verifying that the transaction’s contents are valid, we also need to check that the signatures are valid. A
valid signature over the hash of the transaction prevents tampering.

We can verify that all the transaction’s required signatures are present and valid as follows:

{{< tabs name="tabs-29" >}}
{{% tab name="kotlin" %}}
```kotlin
ySignedTx.verifyRequiredSignatures()

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L525-L525' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 35' end='DOCEND 35' */}}
{{% tab name="java" %}}
```java
    fullySignedTx.verifyRequiredSignatures();

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L542-L542' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 35' end='DOCEND 35' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

However, we’ll often want to verify the transaction’s existing signatures before all of them have been collected. For
this we can use `SignedTransaction.verifySignaturesExcept`, which takes a `vararg` of the public keys for
which the signatures are allowed to be missing:

{{< tabs name="tabs-30" >}}
{{% tab name="kotlin" %}}
```kotlin
SignedTx.verifySignaturesExcept(counterpartyPubKey)

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L532-L532' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 36' end='DOCEND 36' */}}
{{% tab name="java" %}}
```java
    onceSignedTx.verifySignaturesExcept(counterpartyPubKey);

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L549-L549' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 36' end='DOCEND 36' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

If the transaction is missing any signatures without the corresponding public keys being passed in, a
`SignaturesMissingException` is thrown.

We can also choose to simply verify the signatures that are present:

{{< tabs name="tabs-31" >}}
{{% tab name="kotlin" %}}
```kotlin
eSignedTx.checkSignaturesAreValid()

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L539-L539' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 37' end='DOCEND 37' */}}
{{% tab name="java" %}}
```java
    twiceSignedTx.checkSignaturesAreValid();

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L556-L556' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 37' end='DOCEND 37' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

Be very careful, however - this function neither guarantees that the signatures that are present are required, nor
checks whether any signatures are missing.


### Signing the transaction

Once we are satisfied with the contents and existing signatures over the transaction, we add our signature to the
`SignedTransaction` to indicate that we approve the transaction.

We can sign using our legal identity key, as follows:

{{< tabs name="tabs-32" >}}
{{% tab name="kotlin" %}}
```kotlin
twiceSignedTx: SignedTransaction = serviceHub.addSignature(onceSignedTx)

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L401-L401' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 38' end='DOCEND 38' */}}
{{% tab name="java" %}}
```java
SignedTransaction twiceSignedTx = getServiceHub().addSignature(onceSignedTx);

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L414-L414' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 38' end='DOCEND 38' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

Or we can choose to sign using another one of our public keys:

{{< tabs name="tabs-33" >}}
{{% tab name="kotlin" %}}
```kotlin
twiceSignedTx2: SignedTransaction = serviceHub.addSignature(onceSignedTx, otherKey2)

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L406-L406' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 39' end='DOCEND 39' */}}
{{% tab name="java" %}}
```java
SignedTransaction twiceSignedTx2 = getServiceHub().addSignature(onceSignedTx, otherKey2);

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L419-L419' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 39' end='DOCEND 39' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

We can also generate a signature over the transaction without adding it to the transaction directly.

We can do this with our legal identity key:

{{< tabs name="tabs-34" >}}
{{% tab name="kotlin" %}}
```kotlin
sig: TransactionSignature = serviceHub.createSignature(onceSignedTx)

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L416-L416' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 40' end='DOCEND 40' */}}
{{% tab name="java" %}}
```java
TransactionSignature sig = getServiceHub().createSignature(onceSignedTx);

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L429-L429' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 40' end='DOCEND 40' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}

Or using another one of our public keys:

{{< tabs name="tabs-35" >}}
{{% tab name="kotlin" %}}
```kotlin
sig2: TransactionSignature = serviceHub.createSignature(onceSignedTx, otherKey2)

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt#L420-L420' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt' start='DOCSTART 41' end='DOCEND 41' */}}
{{% tab name="java" %}}
```java
TransactionSignature sig2 = getServiceHub().createSignature(onceSignedTx, otherKey2);

```
{{% /tab %}}
{{/* github src='docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' url='https://github.com/corda/corda/blob/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java#L433-L433' raw='https://raw.githubusercontent.com/corda/corda/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java' start='DOCSTART 41' end='DOCEND 41' */}}

[FlowCookbook.kt](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/kotlin/net/corda/docs/FlowCookbook.kt) | [FlowCookbookJava.java](https://github.com/corda/corda/blob/release/os/1.0/docs/source/example-code/src/main/java/net/corda/docs/FlowCookbookJava.java) | ![github](/images/svg/github.svg "github")

{{< /tabs >}}


### Notarising and recording

Notarising and recording a transaction is handled by a built-in flow called `FinalityFlow`. See [API: Flows](api-flows.md) for
more details.
