```sql
CREATE TABLE [dbo].[money_limit](
	[positions_collateral] [float] NOT NULL,
	[openlimit] [float] NOT NULL,
	[wa_position_price] [float] NOT NULL,
	[client_code] [nvarchar](50) NOT NULL,
	[locked_margin_value] [float] NOT NULL,
	[limit_kind] [float] NOT NULL,
	[tag] [nvarchar](50) NOT NULL,
	[leverage] [float] NOT NULL,
	[locked_value_coef] [float] NOT NULL,
	[currentbal] [float] NOT NULL,
	[openbal] [float] NOT NULL,
	[currcode] [nvarchar](50) NOT NULL,
	[orders_collateral] [float] NOT NULL,
	[firmid] [nvarchar](50) NOT NULL,
	[currentlimit] [float] NOT NULL,
	[locked] [float] NOT NULL
) ON [PRIMARY]
```
