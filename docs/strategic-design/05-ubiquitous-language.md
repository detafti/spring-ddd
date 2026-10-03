# 5. Ubiquitous language

The **ubiquitous language** is the vocabulary that domain experts and developers share *inside one bounded context*. It is used in conversations, in documents, and **in the code**: class names, method names, event names and test names. If the experts say "release a shipment to the warehouse", the code says `shipment.releaseToWarehouse()`, not `shipment.setStatus(3)`.

"Ubiquitous" means *everywhere within the context*, not *everywhere in the company*. Each context has its own glossary.

## The same word, different meanings

This table is the main reason Kestrel has several bounded contexts instead of one. Each row was a source of confusion in the Event Storming session.

| Word people use | Ordering | Fulfilment | Inventory | WMS (vendor) | Shipping |
|-----------------|----------|------------|-----------|--------------|----------|
| **Order** | `Order`: what the customer bought, with prices and a promised date | *(not used)* → `Fulfilment`: our plan to deliver one order | *(not used)* | *Outbound order*: one pick task list for one warehouse | *(not used)* |
| **Product** | `OrderLine` with a copied name, SKU and price | `Sku` + quantity to ship | `StockItem`: one SKU in one warehouse | *Item master* record | Parcel contents (weight, dimensions) |
| **Customer** | `Customer` (buyer) | *(not needed)*: only a delivery address | *(not used)* | *(not used)* | `Recipient` |
| **Available** | "In stock" badge shown to shopper | *Promisable*: can be delivered by a date | `AvailableToPromise` = on hand − reserved − safety stock | Physically in a pickable bin | *(not used)* |
| **Shipment** | *(not used)*: the customer sees "your order" | `PlannedShipment`: lines from one warehouse with a promised date | *(not used)* | *(not used)* | `Parcel`: a physical box with a tracking number |
| **Cancel** | `requestCancellation()`: the customer's wish | `acceptCancellation()` / `rejectCancellation()`: the decision | `releaseReservation()` | Cancel outbound order | Void label |

> Note the words marked *(not used)*. Refusing to use a word inside a context is as important as defining it. Fulfilment deliberately never says "order" for its own concept, so nobody confuses its plan with the customer's purchase.

## Glossary per context

### Ordering

| Term | Definition | Code |
|------|------------|------|
| Cart | The shopper's not-yet-committed selection of items. Can change freely. | `Cart` |
| Checkout | The step where the shopper confirms address, sees the promised date, and pays. | `CheckoutService` (application service) |
| Order | A customer's committed purchase. Created when checkout succeeds. Its lines and prices never change afterwards. | `Order` (aggregate root) |
| Order line | One SKU, quantity and agreed unit price on an order. | `OrderLine` (entity inside `Order`) |
| Agreed price | The unit price copied from Catalog at checkout. Not today's price. | `Money` (value object) |
| Promised delivery date | The date shown to the customer at checkout, from a Fulfilment quote. | `PromisedDeliveryDate` (value object) |
| Place an order | Turn a cart into an order. Requires a valid promise quote and an authorised payment. | `Order.place(...)`, event `OrderPlaced` |
| Request cancellation | Record that the customer wants to cancel. Not a guarantee. | `Order.requestCancellation()`, event `CancellationRequested` |
| Order status | The customer-facing progress: `PLACED`, `PARTIALLY_DISPATCHED`, `DISPATCHED`, `DELIVERED`, `COMPLETED`, `CANCELLED`, `BACKORDERED`. | `OrderStatus` (enum) |

### Fulfilment (core)

| Term | Definition | Code |
|------|------------|------|
| Delivery promise | A date we commit to deliver specific lines to a specific postcode. Quoted before checkout, valid for 20 minutes. | `DeliveryPromise` (value object), `PromiseQuote` |
| Fulfilment | Our plan and progress for delivering one order. One per order. | `Fulfilment` (aggregate root) |
| Planned shipment | A group of lines that ship together from one warehouse with one promised date. | `PlannedShipment` (entity inside `Fulfilment`) |
| Sourcing | Choosing which warehouse ships which lines. | `SourcingPolicy` (domain service) |
| Split | Delivering one order in more than one shipment, because no single warehouse has everything. | `Fulfilment.split(...)` |
| Cut-off time | The latest time an order can be released to a warehouse and still leave today. North: 15:00, South: 14:30. | `CutOffTime` (value object) |
| Release | Hand a planned shipment to the warehouse for picking. After release, it cannot be re-planned. | `PlannedShipment.releaseToWarehouse()`, event `ShipmentReleasedToWarehouse` |
| Re-plan | Change the plan after a problem, for example move a shipment to the other warehouse. | `Fulfilment.replan(...)`, event `FulfilmentReplanned` |
| Promise at risk | A problem means the promised date might be missed. Triggers re-planning. | `PromiseRisk` |
| Promise broken | We know we will miss the promised date. The customer must be told. | event `DeliveryPromiseBroken` |
| Back-order | No warehouse can supply a line in a reasonable time. | event `OrderBackordered` |
| Dispatch | The shipment has left the warehouse with the carrier. | event `ShipmentDispatched` |

### Inventory

| Term | Definition | Code |
|------|------------|------|
| SKU | Stock Keeping Unit. The identifier of one sellable item variant. | `Sku` (value object) |
| Stock item | The stock record for one SKU in one warehouse. | `StockItem` (aggregate root) |
| On hand | Units physically in the warehouse according to our books. | `StockItem.onHand` |
| Reservation | Units set aside for a specific fulfilment. Not yet picked. | `Reservation` |
| Safety stock | Units we never promise, to absorb counting errors. | `SafetyStock` |
| Available to promise (ATP) | On hand − reserved − safety stock. The only number Fulfilment uses for promising. | `AvailableToPromise` |
| Stock adjustment | A correction to on hand after a count or damage. | `StockAdjustment`, event `StockAdjusted` |
| Shortage | Fewer units than reserved were found during picking. | event `StockShortageDetected` |

### Catalog

| Term | Definition | Code |
|------|------------|------|
| Product | A thing Kestrel sells, as the shopper sees it ("Cast iron casserole"). | `Product` |
| Variant | One concrete option of a product ("24 cm, red"). Has exactly one SKU. | `Variant` |
| List price | Today's selling price of a variant before promotions. | `ListPrice` |
| Promotion | A time-limited discount rule ("20% off cookware this weekend"). | `Promotion` |
| Publish | Make a product visible in the shop. | event `ProductPublished` |

### Shipping, Payments

| Term | Context | Definition |
|------|---------|------------|
| Parcel | Shipping | A physical box with a tracking number and a carrier. |
| Label | Shipping | The carrier's printed label for a parcel. |
| Recipient | Shipping | Name and address on the label. |
| Authorise | Payments | Reserve the amount on the customer's card at checkout. No money moves. |
| Capture | Payments | Actually take the authorised money, at dispatch. |
| Refund | Payments | Return captured money. |

## Words we banned

| Avoid | Why | Say instead |
|-------|-----|-------------|
| "Status" on its own | Every context has a different one | `OrderStatus`, or name the event that happened |
| "Process the order" | Means something different to every team | Name the step: place, plan, release, dispatch |
| "Item" | Product? Line? Stock? Bin item? | `OrderLine`, `Sku`, `StockItem` |
| "Data", "info", "manager", "handler" in domain class names | Says nothing about the domain | A domain noun or verb |
| Vendor terms (wave, LPN, status 40) outside the ACL | Leaks the WMS model into ours | Our event names |

## How to keep the language alive

- **Code review checks language.** A pull request that adds `OrderManager.updateStatus(int)` to Ordering should be questioned: what actually happened in the business?
- **Tests read like the glossary.** `shouldRejectCancellationAfterShipmentReleasedToWarehouse()` documents a rule better than any wiki.
- **When the language changes, rename the code.** If experts start saying "slot" instead of "promised date", rename the class. A gap between spoken and coded language is technical debt that grows.

Next: [design decisions and trade-offs](06-design-decisions.md).
