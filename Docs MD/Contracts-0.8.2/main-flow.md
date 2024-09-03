# Main Flow

### Deposit

To start sending and accepting INTENTs, the user needs to deposit collateral into the SYMMIO contracts.

**Allocate:** After depositing collateral, the user cannot trade with it yet. It is required to allocate that money to a subaccount, then he will be able to start trading, subaccounts are isolated instances, as SYMMIO is 100% economical sound, all PartyA <> PartyB instances are isolated in subaccounts. All collateral deposited into a subaccount is in cross with all positions opened using that subaccount. To have an isolated position, a seperate subaccount should be created. Subaccounts also enable further customization down the road, where collateral could be allocated to a specific contract instance, with customized code written by PartyA or PartyB.

## Send Quote (Send INTENT)

To start the process of opening a trade, he chooses to open long or short positions by giving some criteria\* & by calling `sendQuote` function.\
\*Criteria: symbol, price, …

## Party B sees the position request

At this time, Party B will see the request, and he can decide whether to accept or ignore it. If he accepts it, first he locks(reserves) the quote through `lockQuote` and then calls `openPosition`. At the time of locking(reserving) the quote before opening the position, no other party Bs can accept the quote until it gets unlocked for any possible reason. (user requests to cancel or Party B cannot open the position)

Reserving Quotes is a "training wheel" feature to make it easier for MarketMakers to respond to quotes, by giving them the ability to reserve a quote, then fulfill their needed hedging operations and then fill the quote afterwards. It could be removed in later versions.

**Cancel Quote:** It is possible that the user wants to cancel his quote(intent) before it gets opened. If the quote isn’t locked(reserved), then it will be canceled immediately. But if it’s locked and isn’t opened yet, depending if it’s partially filled or not, the quote will be canceled. (If it was partially filled already, the unfilled part will be canceled, and the partially filled will be opened.)

## Open Position

After locking(reserving) a quote if nothing unusual happens, the position will be opened by PartyB calling `openPosition`

## Close Request

The PartyA (user) can close the position whenever they want to. This will happen by calling `requestToClosePosition`, Please remark that PartyBs are not able to request to close a position.

## Cancel Close Request

It is possible for the user to cancel their close request (or when using limit order for closing if he wants to make any changes to his order) In these cases if the close request isn’t filled yet, it will be canceled and the position status would change from close pending to a regular position. But if the close request was already partially filled, the already filled portion will remain closed and the remaining amount will be added back to his position and re-opened.
