# Copy Trading Feature

This document describes the copy trading functionality that allows trades from one account to be automatically copied to multiple follower accounts.

## Overview

Copy trading enables a main trading account to have its trades automatically replicated on multiple follower accounts. This is useful for:

- Portfolio management across multiple accounts
- Risk distribion
- Multi-account trading strategies
- Social trading platforms

## Architecture

### Components

1. **Appwrite Database**: Stores API tokens for follower accounts
2. **Copy Trading Service**: Manages token storage and retrieval
3. **Purchase Logic**: Modified to execute trades on multiple accounts
4. **Configuration**: Controls feature enablement and limits

### Database Schema

**Collection: `copy-trading-tokens`**

| Field               | Type    | Required | Description                    |
| ------------------- | ------- | -------- | ------------------------------ |
| `userId`            | string  | Yes      | Main account ID                |
| `followerAccountId` | string  | Yes      | Follower account ID            |
| `apiToken`          | string  | Yes      | API token for follower account |
| `isActive`          | boolean | Yes      | Whether token is active        |
| `createdAt`         | string  | Yes      | Creation timestamp             |
| `updatedAt`         | string  | Yes      | Last update timestamp          |

## Setup

### 1. Install Dependencies

```bash
npm install appwrite
```

### 2. Environment Variables

Add to your `.env` file:

```env
# Appwrite Configuration
APPWRITE_ENDPOINT=https://cloud.appwrite.io/v1
APPWRITE_PROJECT_ID=your-project-id
APPWRITE_DATABASE_ID=trading-bot-db

# Copy Trading Configuration
COPY_TRADING_ENABLED=true
MAX_COPY_TRADING_FOLLOWERS=10
COPY_TRADE_DELAY_MS=1000
COPY_TRADING_LOG_LEVEL=info
```

### 3. Database Setup

Run the setup script to create required collections:

```bash
node scripts/setup-appwrite-collections.js
```

This will create:

- Database: `trading-bot-db`
- Collection: `copy-trading-tokens`
- Required indexes for efficient queries

## Usage

### Enable Copy Trading for a User

```typescript
import { setupCopyTradingForUser } from './utils/copyTradingExamples';

await setupCopyTradingForUser('CR123456', [
    { accountId: 'CR789012', apiToken: 'follower_token_1' },
    { accountId: 'CR345678', apiToken: 'follower_token_2' },
]);
```

### Check Copy Trading Status

```typescript
import { checkCopyTradingStatus } from './utils/copyTradingExamples';

const status = await checkCopyTradingStatus('CR123456');
console.log('Enabled:', status.isEnabled);
console.log('Followers:', status.followerCount);
```

### Remove Follower Account

```typescript
import { removeFollowerAccount } from './utils/copyTradingExamples';

await removeFollowerAccount('CR123456', 'CR789012');
```

## How It Works

### Trade Execution Flow

1. **Main Trade**: User places trade on main account
2. **Validation**: System checks if copy trading is enabled
3. **Token Retrieval**: Fetches active follower tokens from Appwrite
4. **Parallel Execution**: Executes same trade on all follower accounts
5. **Logging**: Records success/failure for each copy trade

### Key Features

- **Non-blocking**: Copy trades don't delay main trade execution
- **Error Isolation**: Failures in copy trades don't affect main trade
- **Configurable**: Enable/disable globally and per user
- **Rate Limiting**: Configurable delays between copy trades
- **Logging**: Comprehensive logging for debugging

## API Reference

### CopyTradingService

#### `storeToken(userId, followerAccountId, apiToken)`

Stores a new follower account token.

#### `getActiveTokens(userId)`

Retrieves all active tokens for a user.

#### `deactivateToken(tokenId)`

Deactivates a token (soft delete).

#### `isCopyTradingEnabled(userId)`

Checks if copy trading is enabled for a user.

## Configuration Options

| Variable                     | Default | Description                          |
| ---------------------------- | ------- | ------------------------------------ |
| `COPY_TRADING_ENABLED`       | `false` | Globally enable/disable copy trading |
| `MAX_COPY_TRADING_FOLLOWERS` | `10`    | Maximum follower accounts per user   |
| `COPY_TRADE_DELAY_MS`        | `1000`  | Delay between copy trades            |
| `COPY_TRADING_LOG_LEVEL`     | `info`  | Logging level                        |

## Security Considerations

1. **Token Storage**: API tokens are stored encrypted in Appwrite
2. **Access Control**: Users can only manage their own follower accounts
3. **Rate Limiting**: Prevents abuse with configurable delays
4. **Audit Logging**: All copy trading activities are logged

## Limitations

1. **API Constraints**: Copy trades depend on available API connections
2. **Rate Limits**: Subject to Deriv API rate limits
3. **Token Validity**: Requires valid API tokens for follower accounts
4. **Network Issues**: Copy trades may fail due to network issues

## Troubleshooting

### Common Issues

1. **Copy trades not executing**
    - Check if `COPY_TRADING_ENABLED=true`
    - Verify follower tokens are active
    - Check Appwrite connectivity

2. **Database connection errors**
    - Verify Appwrite credentials
    - Check network connectivity
    - Ensure database/collection exists

3. **Token validation errors**
    - Ensure tokens are valid Deriv API tokens
    - Check account permissions

### Debug Logging

Set `COPY_TRADING_LOG_LEVEL=debug` for detailed logging.

## Future Enhancements

- Real-time copy trade status monitoring
- Partial copy amounts (percentage of main trade)
- Custom copy trade logic per follower
- Webhook notifications for copy trade events
- Analytics dashboard for copy trading performance
