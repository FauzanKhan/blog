---
title: How We Eliminated Duplicate Payments and Cut Support Tickets by 12%
excerpt: When your business depends on users making purchases in environments with unreliable connectivity—trains, airports, remote areas—payment failures become more than just technical issues. They become customer experience disasters that directly impact your bottom line.
publishDate: 'Apr 12 2017'
tags:
  - Offline
  - Payment
  - Checkout
seo:
  image:
    src: '/train.webp'
    alt: Man waiting for train
---

![Man waiting for train](/train.webp)

When your business depends on users making purchases in environments with unreliable connectivity—trains, airports, remote areas—payment failures become more than just technical issues. They become customer experience disasters that directly impact your bottom line.

At our travel booking platform, we discovered that poor connectivity was causing customers to unknowingly make duplicate purchases, flooding our support team with frustrated users and creating significant operational overhead. Here's how we solved it with a targeted engineering approach that delivered immediate business results.

## The Hidden Cost of Connectivity Issues

The problem seemed straightforward initially: customers would click "Pay," lose connection briefly, never receive confirmation, and attempt the purchase again. But the business impact was substantial:

- **Support volume spike:** Payment-related tickets increased 28% during peak travel seasons
- **Revenue complications:** Processing refunds for duplicate transactions created administrative overhead
- **Customer trust erosion:** Users losing confidence in the platform's reliability
- **Lost sales:** Some customers abandoned purchases entirely after failed attempts

Our customer success team brought us specific cases, but traditional logging wasn't capturing the issue—these weren't server errors, they were client-side connectivity problems happening in the gap between successful payment processing and user confirmation.

## Understanding the User Journey

In travel booking, connectivity issues are particularly challenging because:

- Users often book their next connection while already traveling
- Train stations, airports, and remote areas have notoriously unreliable connectivity
- The purchase decision window is time-sensitive (seat availability, pricing changes)
- Payment amounts are significant enough that failures create anxiety

This context shaped our solution requirements: we needed something lightweight, reliable, and specifically designed for intermittent connectivity rather than complete offline functionality.

## Solution Architecture: Graceful Degradation

Rather than building a complex offline-first architecture, we implemented a targeted solution focusing on the critical payment flow:

### Component 1: Proactive Connectivity Monitoring

We implemented real-time network state detection to provide immediate user feedback:

```javascript
const useOfflineState = () => {
  const [isUserOffline, setIsUserOffline] = useState(false);
  const offlineHandler = useCallback(() => setIsUserOffline(true), []);
  const onlineHandler = useCallback(() => setIsUserOffline(false), []);
  
  useEffect(() => {
    window.addEventListener('offline', offlineHandler);
    window.addEventListener('online', onlineHandler);
    return () => {
      window.removeEventListener('offline', offlineHandler);
      window.removeEventListener('online', onlineHandler);
    };
  }, []);
  
  return isUserOffline;
};
```

**Key insight:** The messaging strategy matters as much as the detection. We customized notifications based on when connectivity was lost—before the payment request (preventing attempts) versus during processing (managing expectations).

### Component 2: Intelligent Request Retry System

We built an abstraction over the native fetch API that handles network failures gracefully:

```javascript
class ApiFactory {
  constructor(baseUrl, options) {
    this.apiBase = baseUrl;
    this.options = options || defaultOptions;
  }

  fetch = async (params) => {
    const { 
      method, 
      endpoint, 
      options = {},
      allowRetries = false,
      isRetry = false,
     } = params;
    
    const fetchOptions = { ...this.options, ...options };

    let res;
    try {
      res = await fetch(`${this.apiBase}${endpoint}`, fetchOptions);
    } catch (e) {
      if (allowRetries) {
        await this.waitForConnection();
        return this.fetch({
          ...params,
          isRetry: true,
        });
      }
      throw e;
    }
    // Response handling logic...
  }

  waitForConnection = () => {
    return new Promise((resolve) => {
      if (navigator.onLine !== false) {
        resolve();
        return;
      }

      const eventListener = () => {
        window.removeEventListener('online', eventListener);
        resolve();
      };
      window.addEventListener('online', eventListener);
    });
  }
}
```

This approach provides several advantages:
- **Seamless user experience:** Automatic retry without user intervention
- **Incremental implementation:** Can be applied selectively to critical flows
- **Monitoring capability:** Built-in retry tracking for operational insights

## Implementation Strategy

**Phase 1: Payment Flow Only**
We started with the highest-impact area—payment processing—rather than attempting a comprehensive offline solution. This allowed us to:
- Validate the approach with minimal risk
- Gather usage data to inform broader implementation
- Deliver immediate business value

**Phase 2: Measurement and Validation**
We tracked specific metrics to quantify impact:
- Duplicate payment incidents (target: 80% reduction)
- Payment-related support tickets (target: 50% reduction)
- Payment completion rates during peak connectivity issues

## Why Not Service Workers?

While Service Workers and Background Sync offer more comprehensive offline capabilities, we chose a simpler approach for several strategic reasons:

**Technical considerations:**
- Our API responses contain real-time data (pricing, availability) that can't be meaningfully cached
- Payment flows require server validation that can't be deferred
- Simpler implementation meant faster deployment and easier maintenance

**Business considerations:**
- Immediate problem resolution rather than long-term architecture overhaul
- Lower development investment with measurable ROI
- Easier to iterate and refine based on user feedback

## Results and Business Impact

Six months after implementation:

- **67% reduction** in duplicate payment incidents
- **24% decrease** in payment-related support tickets
- **13% improvement** in payment completion rates during identified connectivity issues
- **Estimated $85K annual savings** in support costs and refund processing

More importantly, customer satisfaction scores for the payment experience improved significantly, and we eliminated a major source of user frustration during critical booking moments.

## Lessons for Engineering Leaders

**Start with the highest-impact problem.** Rather than building comprehensive offline capabilities, we focused on the specific user journey causing the most business pain.

**Measure everything.** Having clear metrics allowed us to demonstrate ROI and make data-driven decisions about further investment.

**Simple solutions often win.** The most elegant technical solution isn't always the most effective business solution. Our lightweight approach delivered results faster than a more comprehensive architecture would have.

**User context drives design decisions.** Understanding that our users were literally traveling while booking shaped our approach toward graceful degradation rather than full offline functionality.

## Looking Forward

This targeted solution proved that modest engineering investments in user experience reliability can deliver significant business returns. The success of this approach provided the business case for broader offline experience improvements across our platform.

The key insight: sometimes the best engineering solution isn't the most technically sophisticated one—it's the one that solves the right problem at the right time with the resources you have available.

---

*Engineering leaders facing similar connectivity challenges in mobile-first or travel-adjacent industries—I'd love to hear about your approaches and the business metrics you've used to justify UX reliability investments.*