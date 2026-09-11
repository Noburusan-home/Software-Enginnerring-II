## Data Model Requirements

## Data Model Requirements

### `bus_route` table
| Field | Type | Rules |
|-------|------|-------|
| `id` | INTEGER PK | Auto-increment |
| `route_name` | VARCHAR(50) | Required |
| `driver_name` | VARCHAR(100) | Required |
| `sub_driver_name` | VARCHAR(100) | Optional |
| `departure_time` | TIME | Required |
| `arrival_time` | TIME | Required |