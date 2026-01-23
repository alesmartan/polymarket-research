# Mobile Strategy
## Prediction Market Platform

**Version:** 1.0
**Last Updated:** January 2026
**Status:** Active
**Owner:** Product & Engineering Team

---

## Table of Contents

1. [Native vs PWA vs React Native Analysis](#1-native-vs-pwa-vs-react-native-analysis)
2. [Platform Priorities](#2-platform-priorities)
3. [Feature Parity Considerations](#3-feature-parity-considerations)
4. [Push Notification Strategy](#4-push-notification-strategy)
5. [Mobile-Specific UX Patterns](#5-mobile-specific-ux-patterns)
6. [App Store Optimization](#6-app-store-optimization)
7. [Performance Targets](#7-performance-targets)

---

## 1. Native vs PWA vs React Native Analysis

### Executive Summary

Based on our product requirements, user personas, and market analysis, we recommend a **phased approach**:

| Phase | Approach | Timeline | Rationale |
|-------|----------|----------|-----------|
| **Phase 1** | Progressive Web App (PWA) | Months 1-6 | Fastest to market, single codebase |
| **Phase 2** | React Native Apps | Months 7-12 | Enhanced UX, push notifications, App Store presence |
| **Phase 3** | Native Enhancement | Months 13+ | Performance-critical features only |

### Detailed Technology Comparison

#### Progressive Web App (PWA)

| Aspect | Rating | Details |
|--------|--------|---------|
| **Development Speed** | ★★★★★ | Single codebase serves all platforms |
| **Cost** | ★★★★★ | 50-70% less than native development |
| **Performance** | ★★★☆☆ | Good, but not native-level for animations |
| **Device Access** | ★★★☆☆ | Limited biometrics, no push (iOS Safari) |
| **App Store Presence** | ★☆☆☆☆ | No App Store listing (discoverability issue) |
| **Update Speed** | ★★★★★ | Instant updates, no app store approval |
| **Offline Support** | ★★★★☆ | Service workers enable caching |

**PWA Pros for Prediction Markets:**
- Instant deployment of odds updates
- No app store approval delays for market changes
- Single codebase reduces development overhead
- Works across all devices with a browser
- Easy A/B testing and iteration
- Lower user friction (no download required)

**PWA Cons for Prediction Markets:**
- iOS Safari lacks push notification support (until iOS 16.4+)
- Limited background processing for price alerts
- Less discoverable than app store apps
- Cannot access advanced biometrics on some devices
- Perception of being "less legitimate" than native apps

---

#### React Native

| Aspect | Rating | Details |
|--------|--------|---------|
| **Development Speed** | ★★★★☆ | 80% code sharing between platforms |
| **Cost** | ★★★★☆ | 30-40% less than fully native |
| **Performance** | ★★★★☆ | Near-native, occasional bridging overhead |
| **Device Access** | ★★★★★ | Full access to native APIs |
| **App Store Presence** | ★★★★★ | Full App Store and Play Store listing |
| **Update Speed** | ★★★☆☆ | OTA updates + app store reviews |
| **Offline Support** | ★★★★★ | Full native capabilities |

**React Native Pros for Prediction Markets:**
- Cross-platform with single JavaScript codebase
- Access to native device features (biometrics, push)
- App Store presence builds trust and discoverability
- Strong ecosystem (Expo simplifies development)
- Hot reloading speeds development
- Can share code with web (React Native Web)

**React Native Cons for Prediction Markets:**
- Bridge overhead for performance-critical operations
- Some native modules require platform-specific code
- App store review process adds deployment latency
- Larger app size than PWA
- Need to maintain compatibility across RN versions

**React Native Case Study: Polymarket**

> *"Polymarket built a US map with pinch-to-zoom interaction and experimented to make sure no page gestures clashed or were confusing for the user. They worked with React Native libraries but also built custom gestures so that the user can zoom and pan the map without accidentally scrolling the page."*

Polymarket's success with React Native demonstrates viability for prediction markets, achieving a native-quality experience while maintaining development efficiency.

---

#### Native (Swift/Kotlin)

| Aspect | Rating | Details |
|--------|--------|---------|
| **Development Speed** | ★★☆☆☆ | Separate codebases for each platform |
| **Cost** | ★★☆☆☆ | 2x development cost |
| **Performance** | ★★★★★ | Best possible performance |
| **Device Access** | ★★★★★ | Complete platform capabilities |
| **App Store Presence** | ★★★★★ | Full App Store and Play Store listing |
| **Update Speed** | ★★☆☆☆ | App store reviews only |
| **Offline Support** | ★★★★★ | Full native capabilities |

**Native Pros for Prediction Markets:**
- Absolute best performance for charts and animations
- First access to new platform features
- Optimal memory management
- Best-in-class user experience

**Native Cons for Prediction Markets:**
- Double development effort and cost
- Slower time to market
- Feature divergence between platforms
- Harder to maintain consistency

---

### Recommendation: Phased Approach

```
Month 1-6: PWA
├── Rapid market validation
├── Single codebase efficiency
├── Instant updates
└── Foundation for React Native Web later

Month 7-12: React Native
├── iOS and Android apps
├── Push notifications
├── App Store presence
├── Enhanced performance
└── Share code with PWA where possible

Month 13+: Selective Native
├── Performance-critical charts
├── Platform-specific features
└── Competitive differentiation
```

### Technology Stack Recommendation

```
PWA Phase:
├── Framework: Next.js 14+ (App Router)
├── State: Zustand + React Query
├── Styling: Tailwind CSS
├── Charts: Lightweight Charts (TradingView)
├── Service Worker: Workbox
└── PWA: next-pwa

React Native Phase:
├── Framework: React Native + Expo (SDK 50+)
├── Navigation: React Navigation 6
├── State: Zustand + React Query (shared)
├── Charts: react-native-wagmi-charts
├── Push: Expo Notifications + OneSignal
├── Auth: Expo SecureStore
└── Wallet: WalletConnect + wagmi/viem
```

---

## 2. Platform Priorities

### Market Analysis

| Platform | Global Share | Crypto User Share | Priority |
|----------|-------------|-------------------|----------|
| iOS | 27% | 45% | **P0 - Primary** |
| Android | 72% | 55% | **P0 - Primary** |
| Mobile Web | 100% | N/A | **P0 - Foundation** |

### iOS Priority Rationale

**Primary Platform (P0)**

1. **Higher ARPU:** iOS users spend 2.5x more than Android users in finance apps
2. **Crypto Demographics:** Polymarket and Kalshi user research shows iOS overrepresentation
3. **Trust Signal:** iOS App Store listing builds legitimacy
4. **Target Personas:** Sharp traders and institutional users skew iOS

**iOS-Specific Considerations:**
- Strict App Store guidelines for trading apps
- Crypto apps face additional scrutiny
- Face ID/Touch ID integration critical
- Apple Pay for deposits
- Widgets for home screen prices

### Android Priority Rationale

**Primary Platform (P0)**

1. **Volume:** 72% global market share = larger potential user base
2. **Emerging Markets:** Key for international expansion
3. **Flexibility:** Less restrictive app policies
4. **Crypto-Native:** Many crypto users on Android

**Android-Specific Considerations:**
- Google Play policy compliance for gambling/trading
- Device fragmentation (test on 20+ devices)
- Background processing more flexible
- Material Design 3 expectations
- Google Pay integration

### Platform Launch Timeline

```
Month 1-3: PWA (All Platforms)
└── iOS Safari, Android Chrome, Desktop

Month 4-6: PWA Enhancement
├── iOS: Add to Home Screen optimization
├── Android: TWA (Trusted Web Activity) exploration
└── Both: Offline capabilities

Month 7-9: React Native MVP
├── iOS: TestFlight beta
├── Android: Internal testing track
└── Core trading functionality

Month 10-12: App Store Launch
├── iOS: App Store submission
├── Android: Play Store submission
└── Marketing campaigns

Month 13+: Platform Optimization
├── iOS: Widgets, Live Activities
├── Android: Widgets, Quick Settings tile
└── Both: Wear OS / watchOS apps (future)
```

---

## 3. Feature Parity Considerations

### Feature Matrix by Platform

| Feature | Web | PWA | React Native | Priority |
|---------|-----|-----|--------------|----------|
| **Core Trading** |
| Browse markets | ✓ | ✓ | ✓ | P0 |
| Place orders | ✓ | ✓ | ✓ | P0 |
| View portfolio | ✓ | ✓ | ✓ | P0 |
| Price charts | ✓ | ✓ | ✓ | P0 |
| Order book | ✓ | ✓ | ✓ | P0 |
| **Authentication** |
| Email/Password | ✓ | ✓ | ✓ | P0 |
| Social login | ✓ | ✓ | ✓ | P0 |
| Wallet connect | ✓ | ✓ | ✓ | P0 |
| Biometrics | ✗ | Limited | ✓ | P1 |
| **Deposits/Withdrawals** |
| Card payments | ✓ | ✓ | ✓ | P0 |
| Apple/Google Pay | Limited | Limited | ✓ | P1 |
| Crypto deposits | ✓ | ✓ | ✓ | P0 |
| **Notifications** |
| In-app | ✓ | ✓ | ✓ | P0 |
| Push | ✗ | Android only | ✓ | P1 |
| Email | ✓ | ✓ | ✓ | P0 |
| **Advanced** |
| Offline mode | ✗ | Basic | Full | P2 |
| Widgets | ✗ | ✗ | ✓ | P2 |
| Deep linking | Limited | Limited | ✓ | P1 |
| Share sheets | Basic | Basic | Native | P1 |

### Feature Parity Strategy

**Phase 1: Core Parity (MVP)**
All platforms must support:
- Account creation and authentication
- Market browsing and search
- Basic trading (market orders)
- Portfolio viewing
- Deposit via card

**Phase 2: Enhanced Parity**
Native apps add:
- Biometric authentication
- Push notifications
- Native sharing
- Deep linking

**Phase 3: Platform-Specific Excellence**
Each platform leverages unique capabilities:
- iOS: Widgets, Live Activities, Shortcuts
- Android: Widgets, Quick Settings, Bubbles
- Web: Desktop notifications, PWA install

### Handling Feature Gaps

```
┌─────────────────────────────────────────────────────────────────┐
│                    FEATURE GAP HANDLING                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Feature unavailable on platform?                               │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                                                         │   │
│  │  1. GRACEFUL DEGRADATION                               │   │
│  │     └── Feature works differently but achieves goal    │   │
│  │     └── Example: SMS instead of push on iOS Safari     │   │
│  │                                                         │   │
│  │  2. CLEAR MESSAGING                                    │   │
│  │     └── Explain why feature unavailable               │   │
│  │     └── Suggest alternative (download app)             │   │
│  │                                                         │   │
│  │  3. PROGRESSIVE ENHANCEMENT                            │   │
│  │     └── Add feature when platform supports it          │   │
│  │     └── Example: iOS 16.4+ web push                    │   │
│  │                                                         │   │
│  └─────────────────────────────────────────────────────────┘   │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Push Notification Strategy

### Notification Categories

#### 1. Transactional Notifications (Critical)

**Order Notifications:**
```
┌─────────────────────────────────────┐
│ 🔔 Order Filled                     │
│                                     │
│ Your order for 100 "Yes" shares    │
│ in "Will X win?" filled at 65¢     │
│                                     │
│ [View Position]    [Dismiss]        │
└─────────────────────────────────────┘
```

**Deposit/Withdrawal:**
```
┌─────────────────────────────────────┐
│ ✓ Deposit Received                  │
│                                     │
│ $100.00 has been added to your     │
│ account and is ready to trade.      │
│                                     │
│ [Trade Now]        [Dismiss]        │
└─────────────────────────────────────┘
```

**Settings:** Always enabled, cannot be disabled

---

#### 2. Market Notifications (High Engagement)

**Price Alerts:**
```
┌─────────────────────────────────────┐
│ 📈 Price Alert Triggered            │
│                                     │
│ "Will X win?" just hit 70%         │
│ (+5% from your alert at 65%)        │
│                                     │
│ [View Market]      [Dismiss]        │
└─────────────────────────────────────┘
```

**Position Updates:**
```
┌─────────────────────────────────────┐
│ 📊 Position Update                  │
│                                     │
│ Your position in "BTC > $100K" is  │
│ now +$15.00 (+12% today)           │
│                                     │
│ [View Portfolio]   [Dismiss]        │
└─────────────────────────────────────┘
```

**Market Resolution:**
```
┌─────────────────────────────────────┐
│ 🏆 Market Resolved!                 │
│                                     │
│ "Will X win?" resolved YES          │
│ You won $85.00! 🎉                  │
│                                     │
│ [View Results]     [Share]          │
└─────────────────────────────────────┘
```

**Settings:** Opt-in, user configurable

---

#### 3. Engagement Notifications (Growth)

**Trending Markets:**
```
┌─────────────────────────────────────┐
│ 🔥 Trending Now                     │
│                                     │
│ "Fed rate decision" has 10x        │
│ volume today. See what traders      │
│ are predicting.                     │
│                                     │
│ [Explore]          [Dismiss]        │
└─────────────────────────────────────┘
```

**Personalized Recommendations:**
```
┌─────────────────────────────────────┐
│ 💡 Market You Might Like            │
│                                     │
│ Based on your politics trades,      │
│ check out "Will Y win primary?"     │
│ Currently at 45%.                   │
│                                     │
│ [View Market]      [Dismiss]        │
└─────────────────────────────────────┘
```

**Settings:** Opt-in, frequency controlled

---

#### 4. Social Notifications (Viral Growth)

**Leaderboard Updates:**
```
┌─────────────────────────────────────┐
│ 🏅 Leaderboard Update               │
│                                     │
│ You moved up to #47 on the         │
│ weekly leaderboard! Keep trading.   │
│                                     │
│ [View Leaderboard] [Share]          │
└─────────────────────────────────────┘
```

**Friend Activity:**
```
┌─────────────────────────────────────┐
│ 👥 Friend Activity                  │
│                                     │
│ @trader_alex just made a           │
│ prediction on "Movie X grosses $1B" │
│                                     │
│ [See Prediction]   [Dismiss]        │
└─────────────────────────────────────┘
```

**Settings:** Opt-in, fully optional

---

### Notification Timing Strategy

| Category | Timing | Frequency Cap | Silent Hours |
|----------|--------|---------------|--------------|
| Transactional | Immediate | None | None |
| Price Alerts | Immediate | 1/market/hour | User-defined |
| Position Updates | Batched (4hr) | 1/position/day | 10pm-8am |
| Trending | Peak activity (12pm, 6pm) | 2/day max | 9pm-9am |
| Social | Activity time | 3/day max | 10pm-8am |

### Personalization Engine

```
┌─────────────────────────────────────────────────────────────────┐
│                  NOTIFICATION PERSONALIZATION                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  User Signals:                                                  │
│  ├── Trading activity (frequency, volume, categories)          │
│  ├── Market interests (followed categories)                    │
│  ├── Notification engagement (open rate, click rate)           │
│  ├── Active hours (when they trade most)                       │
│  └── Device type (iOS vs Android behavior differences)         │
│                                                                 │
│  Personalization Rules:                                         │
│  ├── High-value traders → More market opportunities            │
│  ├── Casual users → More trending/social content               │
│  ├── Inactive users → Re-engagement with wins/opportunities    │
│  ├── Power users → Less promotional, more alerts               │
│  └── New users → Educational + trending markets                │
│                                                                 │
│  Optimization:                                                  │
│  ├── A/B test notification copy and timing                     │
│  ├── Track conversion to trade action                          │
│  ├── Monitor opt-out rates by category                         │
│  └── Reduce frequency for low-engagement users                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Technical Implementation

**Push Service Architecture:**
```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Events    │────►│ Notification│────►│   Push      │
│   Service   │     │   Router    │     │   Gateway   │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                               │
                    ┌──────────────────────────┼─────────────────┐
                    │                          │                 │
                    ▼                          ▼                 ▼
            ┌───────────────┐         ┌───────────────┐  ┌───────────────┐
            │     APNs      │         │      FCM      │  │   OneSignal   │
            │   (iOS)       │         │  (Android)    │  │   (Backup)    │
            └───────────────┘         └───────────────┘  └───────────────┘
```

**User Preference Schema:**
```json
{
  "user_id": "uuid",
  "push_preferences": {
    "transactional": {
      "enabled": true,
      "silent_hours": null
    },
    "price_alerts": {
      "enabled": true,
      "silent_hours": { "start": "22:00", "end": "08:00" }
    },
    "position_updates": {
      "enabled": true,
      "frequency": "daily_digest",
      "silent_hours": { "start": "22:00", "end": "08:00" }
    },
    "trending": {
      "enabled": true,
      "max_per_day": 2,
      "categories": ["politics", "crypto"]
    },
    "social": {
      "enabled": false
    }
  },
  "timezone": "America/New_York",
  "device_tokens": [
    { "platform": "ios", "token": "...", "last_seen": "..." }
  ]
}
```

---

## 5. Mobile-Specific UX Patterns

### Navigation Pattern

**Bottom Navigation (Primary)**
```
┌─────────────────────────────────────┐
│                                     │
│                                     │
│         [Main Content Area]         │
│                                     │
│                                     │
├─────────────────────────────────────┤
│ [🏠]    [📊]    [💰]    [🏆]    [👤]│
│ Home   Markets  Trade  Leader  You  │
└─────────────────────────────────────┘
```

**Reasoning:**
- Thumb-friendly bottom navigation
- Max 5 items (cognitive limit)
- Icons + labels for clarity
- Active state clearly indicated

### Touch Interactions

**Swipe Gestures:**
| Gesture | Action | Context |
|---------|--------|---------|
| Swipe right | Go back | Navigation |
| Swipe left | Quick actions | Position list |
| Pull down | Refresh | Lists, portfolio |
| Long press | Context menu | Market cards |

**Touch Targets:**
- Minimum: 44x44 points (Apple HIG)
- Recommended: 48x48 points
- Spacing: 8+ points between targets

### Trading Interface (Mobile-Optimized)

```
┌─────────────────────────────────────┐
│ ←  Will X win 2026?              ⋮ │
├─────────────────────────────────────┤
│                                     │
│          65%                        │
│    ┌────────────────────┐          │
│    │████████████░░░░░░░░│          │
│    └────────────────────┘          │
│                                     │
│ ┌───────────────┬───────────────┐  │
│ │               │               │  │
│ │   BUY YES     │    BUY NO     │  │
│ │     65¢       │      35¢      │  │
│ │               │               │  │
│ └───────────────┴───────────────┘  │
│                                     │
│ Amount           [Market ▼]         │
│ ┌─────────────────────────────────┐│
│ │  $                    100       ││
│ └─────────────────────────────────┘│
│                                     │
│ [$25]  [$50]  [$100]  [$500] [Max] │
│                                     │
│ ────────────────────────────────── │
│ Est. shares:           153.84      │
│ If Yes wins:          +$53.84      │
│ ────────────────────────────────── │
│                                     │
│ ┌─────────────────────────────────┐│
│ │        Slide to Confirm →       ││
│ └─────────────────────────────────┘│
│                                     │
└─────────────────────────────────────┘
```

**Mobile Trading Optimizations:**
1. **Large touch targets** for Yes/No buttons
2. **Quick amount buttons** reduce keyboard usage
3. **Slide to confirm** prevents accidental orders
4. **Persistent summary** shows impact
5. **Bottom-anchored CTA** is thumb-accessible

### Market Cards (Mobile)

```
┌─────────────────────────────────────┐
│ 🏛️ Politics                        │
│                                     │
│ Will X win the 2026 election?      │
│                                     │
│ ┌─────────────────────────────────┐│
│ │ Yes ████████████░░░░░ 65%       ││
│ │ No  ██████░░░░░░░░░░░ 35%       ││
│ └─────────────────────────────────┘│
│                                     │
│ 💰 $1.2M  │  ⏱️ 34d  │  📈 +2.5%   │
│                                     │
│ ┌──────────────┬──────────────────┐│
│ │  Trade       │    Share 🔗      ││
│ └──────────────┴──────────────────┘│
└─────────────────────────────────────┘
```

### Loading States

**Skeleton Screens:**
```
┌─────────────────────────────────────┐
│ ▓▓▓▓▓▓▓▓▓▓▓                        │
│                                     │
│ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓       │
│                                     │
│ ┌─────────────────────────────────┐│
│ │ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓           ││
│ │ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓                 ││
│ └─────────────────────────────────┘│
│                                     │
│ ▓▓▓▓▓  │  ▓▓▓▓  │  ▓▓▓▓▓          │
└─────────────────────────────────────┘
```

**Rules:**
- Always use skeleton screens (not spinners) for content
- Match skeleton to actual content layout
- Animate shimmer left-to-right
- Show within 100ms of request

### Offline Handling

```
┌─────────────────────────────────────┐
│         📡                          │
│                                     │
│    You're Offline                   │
│                                     │
│    Don't worry, your portfolio     │
│    data was saved. Prices will     │
│    update when you're back online.  │
│                                     │
│    Last updated: 2 minutes ago      │
│                                     │
│    [  Retry Connection  ]           │
│                                     │
└─────────────────────────────────────┘
```

**Offline Strategy:**
1. Cache portfolio data locally
2. Queue trades for submission when online
3. Show clear offline indicator
4. Allow browsing cached market data
5. Disable trade execution (with explanation)

---

## 6. App Store Optimization

### App Store Listing Strategy

#### App Name and Subtitle

**iOS App Store:**
```
Name: [Brand] - Prediction Markets
Subtitle: Trade on Real-World Events
```

**Google Play Store:**
```
Name: [Brand] - Prediction Markets
Short Description: Trade on elections, sports, crypto & more
```

**Naming Best Practices:**
- Include primary keyword (prediction markets)
- Keep brand name prominent
- Subtitle/short description adds keywords
- Avoid keyword stuffing

#### App Store Description

**Structure:**
```
OPENING (First 3 lines visible)
─────────────────────────────────
Trade on the outcomes of real-world events. From elections to sports
to crypto prices, put your predictions to the test and profit from
your insights.

FEATURES
─────────────────────────────────
📊 THOUSANDS OF MARKETS
Browse markets on politics, sports, entertainment, science, and more.
New markets added daily.

💰 EASY TRADING
Buy "Yes" or "No" shares in seconds. Simple pricing shows probability
at a glance.

📱 REAL-TIME UPDATES
Watch prices move as news breaks. Get instant notifications on your
positions.

🏆 COMPETE & WIN
Climb the leaderboards, earn badges, and prove your prediction skills.

🔒 SECURE & TRUSTED
Bank-grade security. Your funds are protected.

SOCIAL PROOF
─────────────────────────────────
"The best prediction market app I've used" - TechCrunch
"Finally, a prediction market for everyone" - Forbes

CALL TO ACTION
─────────────────────────────────
Download now and get $10 free to start trading!
```

#### Keywords (iOS)

```
Primary:   prediction market, trading, forecast
Secondary: elections, politics, sports betting, crypto
Tertiary:  odds, probability, news, events
```

**Keyword Research Process:**
1. Analyze competitor keywords (Polymarket, Kalshi)
2. Use App Store Connect Search Ads data
3. Track ranking for target keywords
4. Iterate based on performance

#### Screenshots

**Screenshot Strategy:**
```
Screen 1: Hero shot with value proposition
Screen 2: Market discovery interface
Screen 3: Trading interface
Screen 4: Portfolio with gains highlighted
Screen 5: Social features/leaderboard
```

**Design Guidelines:**
- Device frames (iPhone 15 Pro Max, Pixel 8)
- Clear, readable text overlays
- Show actual UI (not mockups)
- Highlight key features
- A/B test screenshot order

#### App Preview Video

**Video Structure (15-30 seconds):**
```
0-5s:  Hook - "Trade on what you know"
5-15s: Show browsing and finding a market
15-25s: Show placing a trade
25-30s: Show winning position + CTA
```

### App Store Compliance

#### iOS App Store Guidelines

**Potential Concerns:**
- 4.7 - Gambling apps require special approval
- 3.1.1 - In-app purchase requirements
- 5.2.1 - Legal requirements for trading

**Mitigation:**
- Position as "prediction market" not "gambling"
- Comply with regional restrictions
- Implement required disclosures
- Consider applying for gambling license where required

#### Google Play Policy

**Potential Concerns:**
- Real-money gambling policy
- Financial services requirements
- Ads and monetization

**Mitigation:**
- Declare gambling/trading functionality
- Implement age verification
- Add gambling addiction resources
- Comply with regional gambling laws

### ASO Performance Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| App Store Impressions | 100K/month | App Store Connect |
| Conversion Rate | 25%+ | Impressions → Downloads |
| Keyword Rankings | Top 10 for primary | ASO tools |
| Ratings | 4.5+ stars | Store average |
| Review Velocity | 50+ reviews/month | Manual tracking |

### Seasonal ASO Optimization

| Period | Focus | Keywords to Add |
|--------|-------|-----------------|
| Election Season | Politics markets | election 2026, vote, political |
| Super Bowl | Sports | super bowl, NFL, football betting |
| March Madness | Sports | basketball, NCAA, bracket |
| Crypto Bull Run | Crypto markets | bitcoin, crypto price, btc |
| Fed Meetings | Economics | interest rates, inflation, fed |

---

## 7. Performance Targets

### Core Metrics

| Metric | Target | Critical | Measurement |
|--------|--------|----------|-------------|
| **App Launch** | <2s | <4s | Cold start to interactive |
| **Time to First Market** | <3s | <5s | Launch to market visible |
| **Trade Execution** | <500ms | <2s | Button tap to confirmation |
| **Price Update** | <100ms | <500ms | Server to display |
| **Scroll FPS** | 60fps | 30fps | Consistent frame rate |
| **App Size** | <50MB | <100MB | Download size |

### Network Performance

| Scenario | Target | Approach |
|----------|--------|----------|
| 4G LTE | Full experience | Standard loading |
| 3G | Functional trading | Reduced images, deferred content |
| 2G | Read-only mode | Cached data, no real-time |
| Offline | Cached portfolio | Local storage, sync when online |

### Memory & Battery

| Metric | Target | Measurement |
|--------|--------|-------------|
| Memory Usage | <150MB | Background |
| Memory Peak | <300MB | During trading |
| Battery (1hr use) | <10% | Active trading session |
| Background Battery | <2%/hr | Price monitoring |

### Optimization Strategies

**Code-Level:**
```
1. Lazy loading for non-critical screens
2. Image optimization (WebP, lazy load)
3. Virtualized lists for market feeds
4. Debounced price updates
5. Memoization for expensive calculations
```

**Network-Level:**
```
1. GraphQL for efficient data fetching
2. WebSocket for real-time prices
3. Edge caching for static content
4. Request batching
5. Optimistic UI updates
```

**React Native Specific:**
```
1. Hermes engine (always enabled)
2. Use native driver for animations
3. FlatList instead of ScrollView for lists
4. Avoid bridge for hot paths
5. Profile with Flipper
```

### Performance Monitoring

**Tools:**
- React Native: Flipper, React DevTools
- iOS: Xcode Instruments
- Android: Android Studio Profiler
- Crash Reporting: Sentry
- Analytics: Amplitude, Mixpanel

**Alerting Thresholds:**
| Metric | Warning | Critical |
|--------|---------|----------|
| App Crash Rate | >0.5% | >1% |
| ANR Rate (Android) | >0.1% | >0.5% |
| Startup Time P95 | >4s | >6s |
| Trade Error Rate | >1% | >5% |

---

## Appendix

### A. Competitive Mobile Analysis

| App | Platform | Approach | Strengths | Weaknesses |
|-----|----------|----------|-----------|------------|
| Polymarket | iOS, Android | React Native | Native feel, real-time | Large app size |
| Kalshi | iOS, Android | Native | Performance | Longer dev cycles |
| Robinhood | iOS, Android | Native | Polish, animations | Expensive to maintain |
| Coinbase | iOS, Android | React Native | Code sharing | Some jank |

### B. Document History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | Jan 2026 | Initial mobile strategy |

### C. References

- [PWA vs React Native Comparison](https://flatirons.com/blog/pwa-vs-react-native/)
- [Polymarket Mobile Case Study](https://www.lazertechnologies.com/case-studies/polymarket)
- [Mobile Trading UX Patterns](https://iceteasoftware.com/blog/build-a-prediction-market-like-polymarket)
- [Push Notification Best Practices](https://onesignal.com/blog/ways-your-crypto-app-can-use-push/)
- [App Store Optimization for Finance](https://www.gummicube.com/blog/aso-finance-apps)
