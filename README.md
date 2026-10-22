# WhatsApp AI Agent + Google Sheets (n8n starter)

This repository is a starter package. GitHub stores the workflow source and documentation; GitHub does not run the n8n workflow. Import the JSON into an n8n instance (n8n Cloud or self-hosted), add credentials, configure the WhatsApp webhook, and then connect Google Sheets tools before using business data.

## Included
- `workflows/whatsapp-ai-agent-starter.json`: WhatsApp text-message trigger -> normalization -> AI Agent -> WhatsApp reply.
- `sheets/Customers.csv`, `Orders.csv`, `Sales.csv`, `Inventory.csv`: header templates. Import each CSV into a separate tab in one Google spreadsheet.

## Setup
1. Create a private GitHub repository and upload this folder. Keep it private if it contains internal workflow details.
2. In Google Sheets, create a spreadsheet and import the four CSV files as tabs named exactly `Customers`, `Orders`, `Sales`, and `Inventory`.
3. In n8n, import `workflows/whatsapp-ai-agent-starter.json` via Workflows -> Import from File.
4. In the OpenAI Chat Model node, select your OpenAI credential. Do not put API keys into JSON or GitHub.
5. In WhatsApp Trigger and WhatsApp Business Cloud nodes, select the appropriate Meta WhatsApp Cloud credentials. Configure the Meta webhook and verify token in the n8n trigger setup.
6. Test using a test WhatsApp number and a harmless message, then activate the workflow.
7. Add Google Sheets nodes/tools for each tab. The starter deliberately does not pretend Sheets is connected: add Google Sheets tool nodes (or separate tool sub-workflows) and connect them to the AI Agent's `ai_tool` input. Configure the spreadsheet document and tab for each tool.
8. Start with read-only tools: find customer, look up order, read inventory, and read sales. Test with sample data.
9. Add write tools only after you have defined authorization and confirmation rules. For writes, use stable IDs and append-or-update/upsert behavior to avoid duplicate records.

## Suggested tools and access policy
- `lookup_customer`: read Customers; return only the minimum data required.
- `lookup_order`: read Orders by exact order_id or authenticated customer's WhatsApp number.
- `read_inventory`: read Inventory; never guess stock or price.
- `read_sales_summary`: read Sales and calculate totals from actual rows.
- `create_order`: require customer confirmation before writing to Orders.
- `record_sale`: restricted to authorized staff; do not let any WhatsApp sender mark a payment as paid.
- `adjust_inventory`: restricted to authorized staff; log who made the change and when.
- `update_order_status`: verify authorization and confirm the intended order before updating.
- Avoid exposing full customer lists or private contact/address details to arbitrary senders.

## Important limitations
- This is an importable starter workflow, not a fully connected production system. Credentials, webhook verification, spreadsheet IDs/tabs, and Google Sheets tool nodes must be configured in your own n8n instance.
- The workflow currently handles text messages only. Image/audio messages need additional branches.
- Conversation Memory is in-memory and may reset; do not treat it as a permanent database.
- GitHub is version control/storage, not an automation runtime. Keep secrets out of commits and repository history.

## Test checklist
- Incoming text triggers one execution.
- AI reply is sent to the same WhatsApp number.
- Invalid/non-text messages fail safely or are handled by a separate branch.
- Sheets tools read the intended tab only.
- Duplicate order IDs do not create duplicate orders.
- A non-admin sender cannot change stock, payment status, or delete records.
- Error handling and execution logs are reviewed before production use.
