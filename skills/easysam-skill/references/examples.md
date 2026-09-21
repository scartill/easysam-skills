# EasySAM Examples Map

The `easysam` codebase includes 20 reference examples under `example/` demonstrating how to configure each supported AWS resource and feature. Refer to these examples when implementing specific architectures:

| Example Directory | Focus / Feature | Key Resource Types & Syntax |
| --- | --- | --- |
| `example/aoss/` | OpenSearch Serverless (AOSS) | `search:`, vector search collections, AOSS policies |
| `example/functionurl/` | AWS Lambda Function URLs | `function_url: true`, `auth_type`, CORS, response streaming |
| `example/customlayer/` | Lambda Custom Layers | `layers: [arn:aws:lambda:...]` under Lambda definition |
| `example/schedule/` | Scheduled / Cron Lambdas | `schedule: "rate(5 minutes)"` or cron expressions |
| `example/kinesismutltiplebuckets/` | Kinesis Streams & Multi-S3 | `streams:`, `buckets:`, Kinesis Firehose delivery |
| `example/sqstrigger/` | Standard SQS queue + poller | `queues:` (null value), `polls:`, custom authorizer |
| `example/fifoqueue/` | Standard & FIFO SQS queues | `queues:` with `fifo: true`, `deduplication_scope`, `polls`/`send` |
| `example/conditionals/` | Conditional Resources | `!Conditional` with environment and region keys |
| `example/dynamottl/` | DynamoDB Time to Live (TTL) | `ttl:` attribute under table definition |
| `example/userenvvars/` | Global & Local Env Vars | `envvars:`, SSM resolution `{{resolve:ssm:...}}` |
| `example/envvarsexpand/` | Context Expansion | Environment variable expansion (`{{environment}}`) |
| `example/myapp/` | Complete Modular Application | Full `import: [backend]`, DynamoDB, S3, SQS, HTTP API |
| `example/onelambda/` | Minimal Single Lambda | Single-file Lambda project setup |
| `example/onelambda314/` | Python 3.14 Runtime | `python: "3.14"` runtime configuration |
| `example/prismarine/` | Prismarine ORM (TypedDict) | `prismarine:`, `@c.model`, DynamoDB streams trigger |
| `example/prismapydantic/` | Prismarine ORM (Pydantic) | `modelling: pydantic`, Pydantic model CRUD testing |
| `example/prismarineconditionals/` | Prismarine Conditional Tables | `conditional-tables:` with `!Conditional` |
| `example/prismarinettl/` | Prismarine Model TTL | `ttl='ExpireAt'` model decorator |
| `example/plugins/` | EasySAM Custom Plugins | `plugins:`, custom Jinja2 template extensions |
| `example/appwitherrors/` | Validation Error Examples | Invalid YAML schemas for testing validation errors |
