```sql
CREATE TABLE [dbo].[money_limit](
	[currcode] [nvarchar](50) NOT NULL,
	[tag] [nvarchar](50) NOT NULL,
	[firmid] [nvarchar](50) NOT NULL,
	[client_code] [nvarchar](50) NOT NULL,
	[openbal] [float] NOT NULL,
	[openlimit] [float] NOT NULL,
	[currentbal] [float] NOT NULL,
	[currentlimit] [float] NOT NULL,
	[locked] [float] NOT NULL,
	[locked_value_coef] [float] NOT NULL,
	[locked_margin_value] [float] NOT NULL,
	[leverage] [float] NOT NULL,
	[limit_kind] [float] NOT NULL,
	[wa_position_price] [float] NOT NULL,
	[orders_collateral] [float] NOT NULL,
	[positions_collateral] [float] NOT NULL
) ON [PRIMARY]
```
