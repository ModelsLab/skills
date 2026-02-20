---
name: modelslab-billing-subscriptions
description: Manage ModelsLab billing, wallet balance, payment methods, subscriptions, invoices, and coupons programmatically via the Agent Control Plane API.
---

# ModelsLab Billing & Subscriptions

Manage billing, wallet funding, payment methods, subscriptions, and coupons via the Agent Control Plane API.

## When to Use This Skill

- Check wallet balance and fund it
- Set up auto-recharge for wallets
- Manage payment methods (add, set default, remove)
- View and download invoices
- Create, upgrade, cancel, or pause subscriptions
- Validate and redeem coupons
- Update billing information (address, tax ID)

## Authentication

All billing endpoints require a bearer token.

```
Base URL: https://modelslab.com/api/agents/v1
Authorization: Bearer <agent_access_token>
```

Get a token via the `modelslab-account-management` skill or `POST /auth/login`.

## Helper

```python
import requests

BASE = "https://modelslab.com/api/agents/v1"

def headers(token):
    return {"Authorization": f"Bearer {token}"}
```

## Billing Overview

```python
def get_billing_overview(token):
    """Get billing overview — balance, active subscriptions, recent charges."""
    resp = requests.get(f"{BASE}/billing/overview", headers=headers(token))
    return resp.json()["data"]

# Usage
overview = get_billing_overview(token)
print(f"Wallet balance: ${overview.get('wallet_balance', 0)}")
```

## Payment Methods

### List Payment Methods

```python
def list_payment_methods(token):
    """List all saved payment methods."""
    resp = requests.get(f"{BASE}/billing/payment-methods", headers=headers(token))
    return resp.json()["data"]
```

### Add Payment Method

```python
def add_payment_method(token, payment_method_id, make_default=True):
    """Add a Stripe payment method.

    Args:
        payment_method_id: Stripe PaymentMethod ID (pm_...)
        make_default: Set as default payment method
    """
    resp = requests.post(
        f"{BASE}/billing/payment-methods",
        headers=headers(token),
        json={
            "payment_method_id": payment_method_id,
            "make_default": make_default
        }
    )
    return resp.json()["data"]
```

### Set Default & Remove

```python
def set_default_payment_method(token, payment_method_id):
    """Set a payment method as the default."""
    requests.put(
        f"{BASE}/billing/payment-methods/{payment_method_id}/default",
        headers=headers(token)
    )

def remove_payment_method(token, payment_method_id):
    """Remove a payment method."""
    requests.delete(
        f"{BASE}/billing/payment-methods/{payment_method_id}",
        headers=headers(token)
    )
```

## Billing Info

```python
def get_billing_info(token):
    """Get billing name, address, and tax ID."""
    resp = requests.get(f"{BASE}/billing/info", headers=headers(token))
    return resp.json()["data"]

def update_billing_info(token, **kwargs):
    """Update billing information.

    Kwargs: name, email, tax_id, tax_id_type,
            address_line1, address_line2, address_city,
            address_state, address_postal_code, address_country
    """
    resp = requests.put(
        f"{BASE}/billing/info",
        headers=headers(token),
        json=kwargs
    )
    return resp.json()["data"]

# Usage
update_billing_info(
    token,
    name="Acme Corp",
    email="billing@acme.com",
    tax_id="EU123456789",
    tax_id_type="eu_vat",
    address_line1="123 AI Street",
    address_city="San Francisco",
    address_state="CA",
    address_postal_code="94105",
    address_country="US"
)
```

## Invoices

```python
def list_invoices(token):
    """List all invoices."""
    resp = requests.get(f"{BASE}/billing/invoices", headers=headers(token))
    return resp.json()["data"]

def get_invoice(token, invoice_id):
    """Get invoice details."""
    resp = requests.get(
        f"{BASE}/billing/invoices/{invoice_id}",
        headers=headers(token)
    )
    return resp.json()["data"]

def get_invoice_pdf(token, invoice_id):
    """Get invoice PDF download URL."""
    resp = requests.get(
        f"{BASE}/billing/invoices/{invoice_id}/pdf",
        headers=headers(token)
    )
    return resp.json()["data"]
```

## Wallet

### Fund Wallet

```python
def fund_wallet(token, amount, payment_method_id=None):
    """Add funds to wallet.

    Args:
        amount: Amount in USD
        payment_method_id: Stripe PM ID (optional, uses default)
    """
    payload = {"amount": amount}
    if payment_method_id:
        payload["payment_method_id"] = payment_method_id

    resp = requests.post(
        f"{BASE}/wallet/fund",
        headers=headers(token),
        json=payload
    )
    return resp.json()["data"]

# Usage
fund_wallet(token, 50)  # Add $50 using default payment method
```

### Auto-Recharge

```python
def enable_auto_funding(token, auto_charge_amount, charge_threshold):
    """Enable automatic wallet recharge.

    Args:
        auto_charge_amount: Amount to charge (min $5)
        charge_threshold: Balance threshold to trigger charge (min $1)
    """
    resp = requests.put(
        f"{BASE}/wallet/auto-funding",
        headers=headers(token),
        json={
            "auto_charge_amount": auto_charge_amount,
            "charge_threshold": charge_threshold
        }
    )
    return resp.json()["data"]

def disable_auto_funding(token):
    """Disable automatic wallet recharge."""
    requests.delete(f"{BASE}/wallet/auto-funding", headers=headers(token))

# Usage — auto-charge $25 when balance drops below $5
enable_auto_funding(token, auto_charge_amount=25, charge_threshold=5)
```

### Withdraw

```python
def withdraw(token, amount):
    """Withdraw from wallet balance."""
    resp = requests.post(
        f"{BASE}/wallet/withdraw",
        headers=headers(token),
        json={"amount": amount}
    )
    return resp.json()["data"]
```

### Coupons

```python
def validate_coupon(token, coupon_code, purchase_amount=None):
    """Validate a coupon code before redeeming."""
    payload = {"coupon_code": coupon_code}
    if purchase_amount:
        payload["purchase_amount"] = purchase_amount

    resp = requests.post(
        f"{BASE}/wallet/coupons/validate",
        headers=headers(token),
        json=payload
    )
    return resp.json()["data"]

def redeem_coupon(token, coupon_code, payment_method_id=None, purchase_amount=None):
    """Redeem a coupon code."""
    payload = {"coupon_code": coupon_code}
    if payment_method_id:
        payload["payment_method_id"] = payment_method_id
    if purchase_amount:
        payload["purchase_amount"] = purchase_amount

    resp = requests.post(
        f"{BASE}/wallet/coupons/redeem",
        headers=headers(token),
        json=payload
    )
    return resp.json()["data"]

# Usage
info = validate_coupon(token, "WELCOME50")
if not info.get("error"):
    redeem_coupon(token, "WELCOME50")
```

## Subscriptions

### List Subscriptions

```python
def list_subscriptions(token):
    """List all active subscriptions."""
    resp = requests.get(f"{BASE}/subscriptions", headers=headers(token))
    return resp.json()["data"]
```

### Create Subscription

```python
def create_subscription(token, plan_id, success_url=None, cancel_url=None):
    """Start a new subscription. Returns a Stripe checkout URL.

    Args:
        plan_id: The plan ID to subscribe to
        success_url: Redirect after successful checkout
        cancel_url: Redirect if checkout is cancelled
    """
    payload = {"plan_id": plan_id}
    if success_url: payload["success_url"] = success_url
    if cancel_url: payload["cancel_url"] = cancel_url

    resp = requests.post(
        f"{BASE}/subscriptions",
        headers=headers(token),
        json=payload
    )
    return resp.json()["data"]
```

### Update (Upgrade/Downgrade)

```python
def update_subscription(token, subscription_id, new_plan_id):
    """Change a subscription to a different plan."""
    resp = requests.put(
        f"{BASE}/subscriptions/{subscription_id}",
        headers=headers(token),
        json={"new_plan_id": new_plan_id}
    )
    return resp.json()["data"]
```

### Cancel, Pause, Resume

```python
def cancel_subscription(token, subscription_id):
    """Cancel a subscription."""
    resp = requests.post(
        f"{BASE}/subscriptions/{subscription_id}/cancel",
        headers=headers(token)
    )
    return resp.json()["data"]

def pause_subscription(token, subscription_id):
    """Pause a subscription."""
    resp = requests.post(
        f"{BASE}/subscriptions/{subscription_id}/pause",
        headers=headers(token)
    )
    return resp.json()["data"]

def resume_subscription(token, subscription_id):
    """Resume a paused subscription."""
    resp = requests.post(
        f"{BASE}/subscriptions/{subscription_id}/resume",
        headers=headers(token)
    )
    return resp.json()["data"]
```

### Other Subscription Actions

```python
def reset_subscription_cycle(token, subscription_id):
    """Reset the current billing cycle."""
    requests.post(
        f"{BASE}/subscriptions/{subscription_id}/reset-cycle",
        headers=headers(token)
    )

def charge_subscription_amount(token, amount):
    """Charge a custom amount ($5-$10,000)."""
    resp = requests.post(
        f"{BASE}/subscriptions/charge-amount",
        headers=headers(token),
        json={"amount": amount}
    )
    return resp.json()["data"]

def reverse_cancelation(token, subscription_id):
    """Reverse a pending cancellation."""
    requests.post(
        f"{BASE}/subscriptions/{subscription_id}/reverse-cancelation",
        headers=headers(token)
    )

def fix_payment(token, subscription_id):
    """Fix a failed subscription payment."""
    requests.post(
        f"{BASE}/subscriptions/{subscription_id}/fix-payment",
        headers=headers(token)
    )
```

## Common Workflow: Ensure Funded Account

```python
def ensure_account_funded(token, min_balance=10, top_up_amount=50):
    """Check wallet balance and fund if low."""
    overview = get_billing_overview(token)
    balance = overview.get("wallet_balance", 0)

    if balance < min_balance:
        print(f"Balance ${balance} below ${min_balance}, funding ${top_up_amount}...")
        fund_wallet(token, top_up_amount)
        print("Wallet funded!")
    else:
        print(f"Balance OK: ${balance}")

# Usage
ensure_account_funded(token, min_balance=5, top_up_amount=25)
```

## MCP Server Access

These same capabilities are available via the Agent Control Plane MCP server:
- **URL**: `https://modelslab.com/mcp/agents`
- **Tools**: `agent-billing`, `agent-wallet`, `agent-subscriptions`

See: https://docs.modelslab.com/mcp-web-api/agent-control-plane

## API Endpoints Reference

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/billing/overview` | Billing overview |
| GET | `/billing/payment-methods` | List payment methods |
| POST | `/billing/payment-methods` | Add payment method |
| PUT | `/billing/payment-methods/{id}/default` | Set default |
| DELETE | `/billing/payment-methods/{id}` | Remove payment method |
| GET | `/billing/info` | Get billing info |
| PUT | `/billing/info` | Update billing info |
| GET | `/billing/invoices` | List invoices |
| GET | `/billing/invoices/{id}` | Invoice details |
| GET | `/billing/invoices/{id}/pdf` | Invoice PDF URL |
| POST | `/wallet/fund` | Fund wallet |
| PUT | `/wallet/auto-funding` | Enable auto-recharge |
| DELETE | `/wallet/auto-funding` | Disable auto-recharge |
| POST | `/wallet/withdraw` | Withdraw balance |
| POST | `/wallet/coupons/validate` | Validate coupon |
| POST | `/wallet/coupons/redeem` | Redeem coupon |
| GET | `/subscriptions` | List subscriptions |
| POST | `/subscriptions` | Create subscription |
| PUT | `/subscriptions/{id}` | Update subscription |
| POST | `/subscriptions/{id}/cancel` | Cancel |
| POST | `/subscriptions/{id}/pause` | Pause |
| POST | `/subscriptions/{id}/resume` | Resume |
| POST | `/subscriptions/{id}/reset-cycle` | Reset cycle |
| POST | `/subscriptions/charge-amount` | Charge amount |
| POST | `/subscriptions/{id}/reverse-cancelation` | Reverse cancel |
| POST | `/subscriptions/{id}/fix-payment` | Fix payment |

## Best Practices

### 1. Set Up Auto-Recharge
```python
# Avoid running out of credits during generation
enable_auto_funding(token, auto_charge_amount=25, charge_threshold=5)
```

### 2. Check Balance Before Large Jobs
```python
overview = get_billing_overview(token)
if overview["wallet_balance"] < estimated_cost:
    fund_wallet(token, estimated_cost * 1.2)  # 20% buffer
```

### 3. Validate Coupons Before Redeeming
```python
info = validate_coupon(token, code)
# Check discount amount, expiry, etc. before committing
```

## Resources

- **API Documentation**: https://docs.modelslab.com/agents-api/billing-and-wallet
- **MCP Server**: https://docs.modelslab.com/mcp-web-api/agent-control-plane
- **Pricing**: https://modelslab.com/pricing
- **Dashboard**: https://modelslab.com/dashboard

## Related Skills

- `modelslab-account-management` - Account setup and API keys
- `modelslab-model-discovery` - Check usage and find models
- `modelslab-webhooks` - Async operation handling
