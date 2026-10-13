## 12. The Full Request Flow

This is the one-line summary from the source notes, expanded:

> `DNS resolver → gives Global Accelerator IPs → Global Accelerator routes to healthy LBs/nodes`

Step by step:

```
1. User types app.example.com
         │
         ▼
2. DNS lookup hits the nearest Route 53 anycast resolver
         │
         ▼
3. Route 53 returns the two STATIC Global Accelerator anycast IPs
         │
         ▼
4. User's packets head to that IP — BGP delivers them to the
   NEAREST AWS edge location (a few ms away)
         │
         ▼
5. Traffic leaves the public internet and enters the AWS backbone
         │
         ▼
6. Global Accelerator picks the closest HEALTHY regional endpoint
         │
         ▼
7. Request is served by the ALB / NLB / EC2 instance in that region
```

If step 6's chosen region fails, only step 6 changes. Steps 1–5 are untouched, which
is exactly why failover is both fast and invisible.

