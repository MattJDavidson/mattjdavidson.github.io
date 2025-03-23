---
layout: post
title: #Pythong logging for cloud environments
---

Observability becomes more important in Cloud Environments.

Developers can start out building and inherit small, linear systems—CRUD APIs, MVC apps, or LLM prototypes—where basic `print()` or framework logging is sufficient. But when systems start scaling, particularly in distributed architectures, observability becomes harder to manage and easy to overlook.

Cloud platforms like CloudWatch, Datadog, and others offer rich observability features: metrics, traces, logs, profiles, dashboards. But they can get expensive fast. And the signal-to-noise ratio doesn’t always scale with cost or complexity.

The simplest, most scalable, and most developer-friendly approach—especially in line with Observability 2.0 thinking—is to get as much value as possible from structured logging.

Logging is something every developer already understands. With a few tweaks in approach, logs can become the foundation of a powerful observability stack. This article outlines practical logging practices—centered around [**AWS Powertools Logger**](https://docs.powertools.aws.dev/lambda/python/latest/)—that are easy to maintain, cost-effective, and support long-term growth of a cloud-native system.

---

## Use AWS Powertools Logger

Use AWS Powertools Logger (if you’re on Lambda) or any logger that emits JSON-structured logs.

It’s built for this use case: structured logging that integrates cleanly with CloudWatch Insights, X-Ray, and other AWS-native tooling.

```python
from aws_lambda_powertools import Logger

logger = Logger()
```

In other languages or Cloud Providers use a similar tool while considering the logging interface and future changes to logging tools. You'll problably not move Cloud providers but you may change logging tools.

---

## Pass Context in `extra` Fields

Always pass contextual information using the `extra` field. Don’t interpolate values into the log string unless it actually makes the message easier to read.

```python
# ✅ this
logger.info("Order processed", extra={"order_id": order_id, "user_id": user_id})

# ✅ also fine if it improves clarity
logger.info("Order processed %s for user %s", order_id, user_id, extra={"order_id": order_id, "user_id": user_id})

# ❌ never this – loses structure and difficult to query
logger.info(f"Order processed {order_id} for user {user_id}")
```

---

## Consistent Key Names in `extra`

- Use `snake_case` for all keys.
- Stick to consistent, predictable names across services and teams.
- Don’t mix styles (`user_id`, not `userId` or `UserID`).

Using consistent entity ids allows simple querying for particular users or domain events.

```
fields @timestamp, @message
| filter user_id = "rowan.smith"
| sort @timestamp desc
```

---

## Log Message Format

- Simple rules, like start log messages with a capital letter.
- Use spaces between words.
- Don’t overthink it, make the message human and machine readable.

Examples:

- ✅ "Processed order"
- ✅ "Order processed successfully"
- ❌ "processed order."
- ❌ "Order processed successfully."

These are only suggestions, pick something simple and easy to be consistent with. Consider using a consistent style for Domain Events like "user_created", "out_of_stock", but you may wish to include these as a key in your log context rather than the primary human readable message.

```python
logger.info("Order processed", extra={"order_id": order_id, "user_id": user_id, "event": "order_processed"})
```

---

## Log Levels

- Use `INFO` for most logging.
- Use `ERROR` with `exc_info=True` when an exception is raised—this is important for tools like Sentry.
- Use `WARNING` when something non-critical but notable happens.
- Avoid `DEBUG` unless you’re actively troubleshooting something.

```python
try:
    process_order()
except Exception:
    logger.error("Order processing failed", extra={"order_id": order_id}, exc_info=True)
```

---

## Log Actions That Have Happened

Log what has happened, not what’s about to happen. This keeps your logs accurate and trustworthy. Eventually you can treate your logs as a record of events and decisions that have happened in your software flow.

Examples:

- ✅ "Order processed"
- ❌ "Starting to process order"

---

## One Context-Rich Line

Prefer a single, well-structured log entry with all relevant context over multiple lines spread out across the code.

```python
# ✅
logger.info("Order processed", extra={"order_id": "1234", "user_id": "5678", "status": "initiated"})

# ❌ noisy and fragmented
logger.info("Order processed")
logger.info("Order ID: 1234")
logger.info("User ID: 5678")
logger.info("Result written")
```

---

## Clear and Actionable Messages

Keep log messages concise and to the point. Avoid overly verbose sentences or messages that tie too closely to specific class or function names that might change.

```python
# ✅
logger.info("Order processed", extra={"order_id": order_id})

# ❌
logger.info("The order has been processed successfully", extra={"order_id": order_id})
logger.info("Order processed successfully by OrderHandlerV2", extra={"order_id": order_id})
```

---

## Consider Log-Based Metrics Over Explicit Metrics

Explicit metrics (e.g. using Powertools Metrics or direct EMF format) are helpful, but they come with tradeoffs—especially in AWS where metric publishing can introduce synchronous latency and costs.

Log-based metrics—built using `MetricFilters` or log queries—can offer a simpler alternative, especially when logs are already structured.

**Why consider log-based metrics?**

- No additional synchronous code or client calls
- Metrics can evolve alongside logs
- Keeps logic simpler and reduces vendor lock-in

> ⚠️ If you change a log message or field name that a dashboard or alarm depends on, it can break silently.

**Pick what’s right for you:**

- If you need low-latency, precise SLO-driven metrics → emit explicit metrics.
- If you want lower overhead and more flexible instrumentation → build from logs.

---

## Trace Ids, Correlation Ids, and Distributed Logging

Once you’re working across services—or even just across Lambda invocations—you’ll want to trace requests through the system.

**Correlation Id vs Trace Id**

- **Correlation Id**: One ID used to tie all logs and actions together for a given request or flow.
- **Trace Id**: Comes from a tracing system like X-Ray or OpenTelemetry, and provides deeper timing/causality insights.

> You want both in your logs wherever possible.

---

## Propagate Ids Across Services

When your service receives a `correlation_id` or `trace_id` (e.g. from an HTTP header or SQS attribute), pass it along to any downstream systems.

**Common propagation headers:**

- `X-Correlation-Id`
- `traceparent` (for W3C trace context)

For message-based systems (e.g. SNS/SQS):

- Include these as message attributes.

---

## Auto-Include Ids in Lambda

If you’re using AWS Powertools, you can inject correlation Ids automatically based on the incoming event:

```python
from aws_lambda_powertools.logging import correlation_paths

logger = Logger()

@logger.inject_lambda_context(correlation_id_path=correlation_paths.API_GATEWAY_REST)
def handler(event, context):
    logger.info("Handled request")
```

I suggest using tools like **AWS Powertools** to minimise the overhead of propagating trace and correlation ids through your systems. Where possible use them to inject context from queues and APIs. This ensures every log has consistent context without needing to manually wire repeated fields.

---

## Summary

- ✅ Use structured logs with consistent keys
- ✅ Include context using `extra`, not interpolated strings
- ✅ Log actions that happened, not intentions
- ✅ Keep messages readable and focused
- ✅ Include `correlation_id` and `trace_id` in every log
- ✅ Pass these Ids through all downstream calls and systems
- ✅ Prefer a single context-rich log over fragmented noise
- ✅ Consider log-based metrics if it fits your system’s needs better than explicit ones

---

### Next Steps

- Use richer queries to debug issues that may be difficult with traditional logging. Give developers the tools to write context rich queries rather than joining disparate logs and db records together ad hoc.

```
fields @timestamp, user_id, order_id, event
| filter source = "internal_system_x"
    and response = 502
    and event = "order_created"
| sort @timestamp desc
```

- Consider moving towards [Canonical Logging](https://baselime.io/blog/canonical-log-lines).
- Read the baselime (now Cloudflare) [blog.](https://baselime.io/blog) Thanks [Tom](https://bsky.app/profile/ankcorn.dev) for the help when I started with Canonical logging.
- Read [Observability Engineering.](https://www.oreilly.com/library/view/observability-engineering/9781492076438/)

I have a similar post planned for how to incrementally migrate towards wide context/Canonical logs in Python on AWS.
