# Design Notes: Munder Difflin Multi-Agent System

## What this is

A system that handles customer requests for a paper supply company using five agents built with the smolagents framework. One agent acts as a supervisor that reads a customer request and decides which specialist should handle it. The specialists check inventory, generate quotes, process sales, and manage restocking.

## The agents

**CustomerOperationsSupervisor** is the entry point. Every request goes through it first. It reads the request, looks at a set of routing rules in its prompt (keywords like "inventory," "quote," "buy"), and calls one of three tools to hand the request off:

- `delegate_to_inventory_specialist`
- `delegate_to_quote_specialist`
- `delegate_to_sales_transaction_agent`

**InventorySpecialist** answers questions about stock levels and delivery timelines. It has four tools: `get_inventory_status`, `check_inventory_availability`, `get_delivery_timeline`, and `get_all_available_inventory`.

**QuoteSpecialistAgent** builds price quotes. It has two tools: `search_quote_history`, which looks at past quotes for similar requests, and `calculate_quote`, which does the actual math.

**SalesTransactionAgent** handles purchases. It checks whether requested items are in stock with `can_fulfill_order`, then creates orders with `create_sales_order` for whatever is available. It is told explicitly not to invent prices or transaction IDs and to use exactly what `create_sales_order` returns.

**SupplyOperationsSpecialist** decides whether to reorder stock after a sale. It checks whether an item fell below its minimum stock level, checks available cash, and creates a purchase order if needed.

## How a request flows through the system

1. A customer request comes in as plain text, for example: "I need 200 sheets of A4 glossy paper delivered by April 15."
2. The supervisor reads it and picks a specialist based on the wording of the request.
3. If it is a sales request, the SalesTransactionAgent checks each item against current inventory, creates a sales order for anything available, and skips anything that is not.
4. If at least one item sold, the system automatically runs the SupplyOperationsSpecialist afterward to check if that sale pushed any item below its minimum stock level. If so, it creates a purchase order to restock.
5. The result bubbles back up through the supervisor and is returned to the caller as a combined summary of the sale and any restocking decision.

One thing worth noting about this flow: SupplyOperationsSpecialist is never called directly by the supervisor, even though the supervisor's own instructions mention it as an option for "low stock, reorder" requests. In the actual code, the supervisor only has tools to reach the inventory, quote, and sales agents. Supply operations only run as a side effect after a completed sale. A standalone question like "should we reorder cardstock" would not currently reach that agent. This is a gap I would fix by giving the supervisor a fourth delegate tool for supply requests.

## How state is tracked

There is no single variable holding "current cash" or "current stock." Instead, every sale and every stock order gets written as a row in a `transactions` table in a SQLite database. Cash balance and inventory levels are both calculated by adding up the relevant rows in that table up to a given date. This means the system always has a full history of what happened and when, and a snapshot for any date can be reconstructed by replaying the transactions up to that point.

For a project this size, recomputing balances by scanning the transactions table works fine and keeps the logic simple: there is no risk of a stored balance drifting out of sync with the actual history, and every past date can be reconstructed on demand. If this were a production system handling a much larger transaction volume, I would not scan the full table on every read. I would add an index on transaction_date and item_name so lookups are not full table scans, and I would introduce periodic balance snapshots, so a balance is computed by taking the most recent snapshot and only summing the transactions since then instead of replaying the entire history every time. The full transaction log would stay in place either way, since that is what makes past dates reconstructable and the numbers auditable.

## Testing

I ran the system against a sample set of customer requests from `quote_requests_sample.csv`, processing each one in order by request date. The script prints out each agent's reasoning and tool calls as it runs, tracks cash and inventory balances after each request, and writes a summary of all requests to `test_results.csv`. The run included both fully fulfilled orders and cases where an item was out of stock, and the system correctly separated fulfilled items from unavailable ones in its response rather than reporting a sale that did not fully happen.
