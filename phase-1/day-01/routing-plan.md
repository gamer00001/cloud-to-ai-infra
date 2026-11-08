#Route Table
| Route table  | Attached subnets    | Destination   | Target  |
| rt-public    | public-a, public-b  | 10.20.0.0/16  | local   |
|                                    |   0.0.0.0/0   | igw     |
| rt-app-a     | app-a               | 10.20.0.0/16  | local   |
|                                    | 0.0.0.0/0     | nat-a   |
| rt-app-b     | app-b               | 10.20.0.0/16  | local   |
|                                    | 0.0.0.0/0     | nat-b   |
| rt-db        | db-a, db-b          | 10.20.0.0/16  | local   |

## Security groups
| SG     | Direction | Port       | Source/Dest |    Why                     |
| sg-alb | in        | 443, 80    | 0.0.0.0/0   | public HTTPS, 80 redirects |
| sg-alb | out       | 3000       | sg-app      | forward to app             |
| sg-app | in        | 3000       | sg-alb      | only ALB reaches the app   |
| sg-app | out       | 5432, 6379 | sg-db       | Postgres, Redis            |
| sg-app | out       | 443        | 0.0.0.0/0   | patches, APIs (via NAT)    |
| sg-db  | in        | 5432, 6379 | sg-app      | only the app reaches data  |