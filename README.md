# CryptoCloud Payment Integration API

A complete .NET 10 API integration for CryptoCloud cryptocurrency payment gateway with automatic transaction monitoring and postback handling.

## Features

 **Invoice Creation** - Create payment invoices via CryptoCloud API
 **Automatic Status Monitoring** - Background service checks pending transactions
 **Postback Webhook** - Automatic notification handling when payments complete
 **JWT Verification** - Secure postback validation using JWT tokens
 **Database Persistence** - SQLite database for invoice and transaction storage
 **Transaction History** - Track all blockchain transactions for each invoice
 **Payment Statistics** - Real-time payment analytics and reporting
 **Swagger UI** - Interactive API documentation and testing

## Quick Start

### 1. Configuration

Edit `appsettings.json` and add your CryptoCloud credentials:

```json
{
  "CryptoCloud": {
    "ApiKey": "your-api-key-here",
    "ShopId": "your-shop-id-here",
    "SecretKey": "your-secret-key-here"
  }
}
```

**Where to get these values:**
1. Go to [CryptoCloud Dashboard](https://cryptocloud.plus/)
2. Create a project or select existing one
3. Get **API Key** from API section
4. Get **Shop ID** from project settings
5. Get **Secret Key** for JWT verification from project settings
6. Configure **Postback URL** to: `https://yourdomain.com/api/postback/notify`

### 2. Run the Application

```bash
dotnet restore
dotnet run
```

The API will start on `https://localhost:5001` or `http://localhost:5000`

### 3. Open Swagger UI

Navigate to `https://localhost:5001/` to access the interactive API documentation.

![Swagger UI](https://via.placeholder.com/800x400?text=Swagger+UI+Interface)

### 4. Test the Integration

**Using Swagger UI:**
1. Open `https://localhost:5001/`
2. Click on **POST /api/payment/create**
3. Click **Try it out**
4. Edit the request body and click **Execute**

**Using cURL:**

```bash
curl -X POST https://localhost:5001/api/payment/create \
  -H "Content-Type: application/json" \
  -d '{
    "orderId": "ORDER-12345",
    "amount": 100,
    "currency": "USD",
    "email": "customer@example.com"
  }'
```

**Response:**
```json
{
  "success": true,
  "invoice_uuid": "INV-XXXXXXXX",
  "payment_link": "https://pay.cryptocloud.plus/XXXXXXXX",
  "amount": 100,
  "status": "created"
}
```

## API Documentation

### Interactive Documentation (Swagger UI)

Access the full interactive API documentation at:
- **Development:** `https://localhost:5001/`
- **Production:** `https://yourdomain.com/swagger`

Features:
- ?? Complete endpoint documentation
- ?? Test API calls directly from browser
- ?? Request/response examples
- ?? Download OpenAPI specification
- ?? Search and filter endpoints

See [SWAGGER.md](SWAGGER.md) for detailed Swagger UI guide.

## API Endpoints

### Payment Management

#### Create Invoice
```http
POST /api/payment/create
Content-Type: application/json

{
  "orderId": "ORDER-001",
  "amount": 100,
  "currency": "USD",
  "email": "user@example.com",
  "cryptocurrency": "USDT_TRC20",
  "availableCurrencies": ["USDT_TRC20", "BTC", "ETH"],
  "timeToPayHours": 24,
  "metadata": "{\"userId\": \"123\"}"
}
```

#### Get Invoice by UUID
```http
GET /api/payment/{invoiceUuid}
```

#### Get Invoice by Order ID
```http
GET /api/payment/order/{orderId}
```

#### Refresh Invoice Status
```http
POST /api/payment/{invoiceUuid}/refresh
```

#### Get Statistics
```http
GET /api/payment/statistics
```

### Postback Webhook

#### Receive Payment Notification
```http
POST /api/postback/notify
```

This endpoint is called automatically by CryptoCloud when payment is completed.

#### Test Postback Endpoint
```http
GET /api/postback/test
```

## Background Service

The **InvoiceStatusMonitorService** automatically:
- Checks pending invoices every 60 seconds (configurable)
- Updates invoice status from CryptoCloud API
- Monitors invoices for 24 hours (configurable)
- Handles transaction detection
- Logs all status changes

**Configuration:**
```json
{
  "CryptoCloud": {
    "StatusCheckIntervalSeconds": 60,
    "MonitoringPeriodHours": 24
  }
}
```

## Database Schema

### PaymentInvoices Table
- `Id` - Primary key
- `InvoiceUuid` - CryptoCloud invoice UUID
- `OrderId` - Your internal order ID
- `Amount` - Invoice amount in fiat
- `AmountUsd` - Amount in USD
- `ReceivedAmount` - Actually received amount
- `Currency` - Fiat currency code
- `CryptoCurrency` - Crypto currency code
- `PaymentAddress` - Blockchain address
- `PaymentLink` - Payment page URL
- `Status` - Invoice status (created, paid, partial, overpaid, canceled, expired)
- `CustomerEmail` - Customer email
- `CreatedAt` - Creation timestamp
- `ExpiryDate` - Expiration timestamp
- `PaidAt` - Payment completion timestamp
- `LastStatusCheck` - Last status check timestamp
- `Fee` - Transaction fee
- `ServiceFee` - Service commission
- `TestMode` - Test mode flag
- `Metadata` - Additional JSON data

### PaymentTransactions Table
- `Id` - Primary key
- `PaymentInvoiceId` - Foreign key to PaymentInvoices
- `TransactionHash` - Blockchain transaction hash
- `Amount` - Transaction amount
- `Currency` - Cryptocurrency
- `DetectedAt` - Detection timestamp
- `Confirmations` - Number of confirmations

## Supported Cryptocurrencies

- **Bitcoin**: BTC
- **Ethereum**: ETH, USDT_ERC20, USDC_ERC20, TUSD_ERC20, SHIB_ERC20
- **Ethereum Arbitrum**: ETH_ARB, USDT_ARB, USDC_ARB
- **Ethereum Optimism**: ETH_OPT, USDT_OPT, USDC_OPT
- **Ethereum Base**: ETH_BASE, USDC_BASE
- **Tron**: TRX, USDT_TRC20, USDD_TRC20
- **Binance Smart Chain**: BNB, USDT_BSC, USDC_BSC, TUSD_BSC
- **TON**: TON, USDT_TON
- **Solana**: SOL, USDT_SOL, USDC_SOL
- **Litecoin**: LTC

## Invoice Statuses

- `created` - Invoice created, waiting for payment
- `paid` - Payment received and confirmed
- `partial` - Partial payment received
- `overpaid` - More than expected amount received
- `canceled` - Invoice canceled
- `expired` - Invoice expired without payment

## Security

### JWT Token Verification

All postback notifications are verified using JWT tokens:
1. CryptoCloud sends a JWT token with each postback
2. Token is signed with your secret key using HS256 algorithm
3. Token contains invoice UUID and expiration time (5 minutes)
4. API verifies token signature before processing postback
5. Invalid or expired tokens are rejected

### Best Practices

- ? Keep your `ApiKey` and `SecretKey` secret
- ? Use HTTPS in production
- ? Validate all postback notifications
- ? Check invoice status before delivering goods/services
- ? Monitor background service logs
- ? Set up proper error handling
- ? Use test mode during development

## Testing

### Test Mode

Enable test mode in configuration:
```json
{
  "CryptoCloud": {
    "TestMode": true
  }
}
```

In test mode:
- No real payments are processed
- Invoices are marked as test
- Full API functionality available
- Perfect for development and testing

### Testing Postback

1. Use [ngrok](https://ngrok.com/) or similar for local testing:
   ```bash
   ngrok http 5001
   ```

2. Configure postback URL in CryptoCloud dashboard:
   ```
   https://your-ngrok-url.ngrok.io/api/postback/notify
   ```

3. Create test invoice and make payment

4. Check logs for postback reception

## Monitoring & Logs

The application provides detailed logging:

```
[Information] Invoice created: INV-XXXXXXXX for order ORDER-001
[Information] Starting invoice status check cycle
[Information] Found 5 pending invoice(s) to check
[Information] Invoice INV-XXXXXXXX status changed: created -> paid
[Information] Received postback for invoice INV-XXXXXXXX, status success
[Information] Postback processed successfully for invoice INV-XXXXXXXX
```

## Troubleshooting

### Issue: Postback not received

**Solution:**
1. Check postback URL is configured in CryptoCloud dashboard
2. Ensure URL is publicly accessible (use ngrok for local testing)
3. Check firewall rules
4. Review logs for incoming requests

### Issue: JWT verification fails

**Solution:**
1. Verify `SecretKey` matches the one in CryptoCloud dashboard
2. Check system time is synchronized
3. Ensure token hasn't expired (5 minute lifetime)

### Issue: Invoice status not updating

**Solution:**
1. Check background service is running
2. Verify API credentials are correct
3. Check network connectivity to CryptoCloud API
4. Review logs for errors

### Issue: Database errors

**Solution:**
1. Delete `payments.db` file to recreate database
2. Check file permissions
3. Ensure SQLite is properly installed

## Production Deployment

### Requirements

- .NET 10 Runtime
- Public HTTPS endpoint for postback
- SQLite or migrate to SQL Server/PostgreSQL
- Proper logging and monitoring

### Environment Variables

You can override appsettings.json using environment variables:

```bash
export CryptoCloud__ApiKey="your-api-key"
export CryptoCloud__ShopId="your-shop-id"
export CryptoCloud__SecretKey="your-secret-key"
```

### Database Migration

For production, consider using SQL Server or PostgreSQL:

```csharp
// In Program.cs, replace:
options.UseSqlite("Data Source=payments.db")

// With:
options.UseSqlServer(connectionString)
// or
options.UseNpgsql(connectionString)
```

## Documentation Links

- [CryptoCloud Official Documentation](https://docs.cryptocloud.plus/en)
- [API Reference](https://docs.cryptocloud.plus/en/api-reference-v2/authorization)
- [Support](https://support.cryptocloud.plus/en)

## License

MIT License - See LICENSE file for details

## Support

For issues and questions:
- CryptoCloud Support: https://support.cryptocloud.plus/en
- API Documentation: https://docs.cryptocloud.plus/en

---

**Note:** Replace all placeholder values (API keys, shop IDs, etc.) with your actual credentials before deploying to production.
