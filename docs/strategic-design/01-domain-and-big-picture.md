# 1. The domain and the big picture

Strategic design starts with the business, not with tables or services. Before drawing a single box, we need to know **how the company makes money and where it wins against competitors**, because that decides where we spend our modelling effort.

## The business: Kestrel Supply

Kestrel Supply sells home and kitchen goods online (around 8,000 products) and ships them from **two warehouses it runs itself** (North and South). It is not the cheapest shop in its market. Customers choose it because of one promise:

> **"Order by 3 pm, and the date we show you at checkout is the date it arrives."**

Kestrel shows a concrete delivery date on every product page and at checkout, and it keeps that promise more than 97% of the time. Competitors show vague ranges ("3 to 5 working days") or promise dates they miss. Marketing, repeat purchases and the company's pricing power all rest on that promise.

That one sentence is the most important input to everything in this module. Keep it in mind.

## Who is involved

| Person | Cares about |
|--------|-------------|
| Shopper | Finding products, knowing exactly when they will arrive, paying safely |
| Customer service agent | Where is an order, can it still be changed or cancelled |
| Merchandiser | Product data, categories, prices, promotions |
| Fulfilment planner | Which warehouse ships what, cut-off times, keeping the promise when stock is short |
| Warehouse supervisor | Pick waves, staff, getting parcels onto the carrier truck before it leaves |
| Finance | Payments captured correctly, refunds, invoices |

Notice that each of these people uses some of the same words (order, product, available, shipment) to mean different things. That is a strong hint about where boundaries will go. We come back to it in the [glossary](05-ubiquitous-language.md).

## Big-picture Event Storming

Event Storming is a workshop where domain experts and developers put **domain events** (things that happened, written in the past tense) on a timeline. It is the fastest way to discover a domain without arguing about data models. Below is the result of a big-picture session, cleaned up.

### Happy path

```
Merchandising        Shopping & checkout               Fulfilment & warehouse                              Delivery & money
─────────────        ───────────────────               ──────────────────────                              ────────────────
ProductPublished  →  ItemAddedToCart                →  FulfilmentPlanned                                →  ShipmentDispatched
PriceChanged         DeliveryPromiseQuoted             StockReserved                                       PaymentCaptured
PromotionStarted     CheckoutStarted                   ShipmentReleasedToWarehouse                         ParcelInTransit
                     PaymentAuthorised                 ShipmentPicked                                      ParcelDelivered
                     ★ OrderPlaced                     ShipmentPacked                                      ★ OrderCompleted
                                                       ★ ShipmentDispatched
```

### Unhappy paths (where the interesting rules live)

| Event | What it means | Why it matters |
|-------|---------------|----------------|
| `PaymentAuthorisationFailed` | Card declined at checkout | Order is never placed |
| `StockShortageDetected` | Planned stock is missing (damaged, miscounted) | The promise is now **at risk** |
| `FulfilmentReplanned` | Shipment moved to the other warehouse or split | The core trick that keeps the promise |
| `DeliveryPromiseBroken` | We will miss the date we showed the customer | Customer must be told, maybe compensated |
| `OrderBackordered` | No stock anywhere | Customer chooses to wait or cancel |
| `CancellationRequested` | Customer wants to cancel | Only possible before the shipment is picked |
| `CarrierMissedCollection` | Truck left without our parcels | Promise at risk again |
| `ParcelReturned` | Customer sent it back | Refund and restock |

### Pivotal events

Some events mark a hand-over where **the language and the people change**. These are called pivotal events (marked ★ above), and they are the best early clue for boundaries:

- **`OrderPlaced`**: before it, we talk about carts, prices and checkout with the shopper. After it, we talk about warehouses, stock and shipments with planners.
- **`ShipmentDispatched`**: before it, the parcel is ours and inside our warehouse. After it, the carrier owns the movement, and Finance captures the payment.
- **`OrderCompleted`**: the commitment to the customer is closed.

### Hotspots (open questions raised during the session)

Event Storming marks disagreements and unknowns as hotspots. These were the important ones, and the answers shaped the design:

1. **"Who decides whether an order can be cancelled?"** Ordering knows the customer wants to cancel, but only Fulfilment knows whether the shipment has already been picked. Answer: Ordering *requests* cancellation, Fulfilment *decides*. See [decisions](06-design-decisions.md#d6-cancellation-is-a-request-not-a-command).
2. **"Is a product 'available' when it is in stock?"** Merchandisers, planners and warehouse staff all said yes and all meant different things. See the glossary.
3. **"Where is the delivery date calculated?"** It is shown in the shop but depends on stock, warehouse cut-offs and carrier schedules. Answer: Fulfilment calculates it, Ordering displays it.
4. **"Do we capture payment at checkout or at dispatch?"** Card rules say capture at dispatch. So Payments listens to `ShipmentDispatched`.

Next: [classify the subdomains](02-subdomains.md).
