# rustorm
A simple ORM for rust

##Dependency
*rust-postgres

##Features

*intelligent model code generation
	The ability to figure out linker tables, 
	then build 1:M relation with the tables
	The ability to figure out extension tables

###example:	 

```
-- Table: bazaar.product

-- DROP TABLE bazaar.product;

CREATE TABLE bazaar.product
(
-- Inherited from table system.record:  organization_id uuid, -- @Value(users.user_id) , which means the value will be set with the users.user_id value...
-- Inherited from table system.record:  client_id uuid, -- @Value(users.client_id) The client_id of the user creating this records
-- Inherited from table system.record:  created timestamp with time zone NOT NULL DEFAULT now(),
-- Inherited from table system.record:  createdby uuid, -- @Value(users.user_id)
-- Inherited from table system.record:  updated timestamp with time zone NOT NULL DEFAULT now(),
-- Inherited from table system.record:  updatedby uuid, -- @Value(users.user_id)
-- Inherited from table system.record:  priority numeric,
-- Inherited from table system.record:  name character varying, -- This is @Required it has @DisplayLength(50) - 50 character in display length a @MinLength(1) and @MaxLength(100) - Do not go over 100 characters or else the system will throw a ValueTooLong exception...
-- Inherited from table system.record:  description character varying, -- @DisplayLength(100) When building a UI for this field...
-- Inherited from table system.record:  help text,
-- Inherited from table system.record:  active boolean NOT NULL DEFAULT true, -- @Active
  product_id uuid NOT NULL DEFAULT uuid_generate_v4(),
  parent_product_id uuid,
  is_service boolean DEFAULT false,
  price numeric,
  use_parent_price boolean DEFAULT false,
  unit character varying,
  tags json,
  info json, -- {color:"red",...
  seq_no integer, -- @Sequence can be used to do alternate ordering of the values, when alphetical or time can not be used
  upfront_fee numeric DEFAULT 0.00, -- Applicable to services, usually services has an upfront fee
  barcode character varying, -- barcode if scanning the product, conflict can happen, expect to return matching list of products using the barcode
  owner_id uuid, -- Whom this product belongs, since createdby can be someone else create the product list in behalf of the owner of the product
  currency_id uuid,
  CONSTRAINT product_pkey PRIMARY KEY (product_id),
  CONSTRAINT product_currency_id_fkey FOREIGN KEY (currency_id)
      REFERENCES payment.currency (currency_id) MATCH SIMPLE
      ON UPDATE CASCADE ON DELETE RESTRICT DEFERRABLE INITIALLY DEFERRED,
  CONSTRAINT product_user_id_fkey FOREIGN KEY (owner_id)
      REFERENCES bazaar.users (user_id) MATCH SIMPLE
      ON UPDATE CASCADE ON DELETE RESTRICT DEFERRABLE INITIALLY DEFERRED
)
INHERITS (system.record)
WITH (
  OIDS=FALSE
);

```


The generated rust code will be

```rust
///
/// This will be exposed as an @Api, including @Table(users, category, product_availability, photo)
///
pub struct Product {
	/// primary
	/// defaults to: uuid_generate_v4()
	/// not nullable 
	pub product_id:Uuid,
	/// barcode if scanning the product, conflict can happen, expect to return matching list of products using the barcode
	pub barcode:Option<String>,
	pub currency_id:Option<Uuid>,
	/// {color:"red",
	/// dimension:"10x20x30",
	/// dimensionUnit:"mm",
	/// weight:"4",
	/// weightUnit:"kg"
	/// }
	pub info:Option<Json>,
	/// defaults to: false
	pub is_service:Option<bool>,
	/// Whom this product belongs, since createdby can be someone else create the product list in behalf of the owner of the product
	pub owner_id:Option<Uuid>,
	pub parent_product_id:Option<Uuid>,
	pub price:Option<f64>,
	/// @Sequence can be used to do alternate ordering of the values, when alphetical or time can not be used
	pub seq_no:Option<i32>,
	pub tags:Option<Json>,
	pub unit:Option<String>,
	/// Applicable to services, usually services has an upfront fee
	/// defaults to: 0.00
	pub upfront_fee:Option<f64>,
	/// defaults to: false
	pub use_parent_price:Option<bool>,
	/// @Active
	/// defaults to: true
	/// not nullable 
	/// --inherited-- 
	pub active:bool,
	/// @Value(users.client_id) The client_id of the user creating this records
	/// --inherited-- 
	pub client_id:Option<Uuid>,
	/// defaults to: now()
	/// not nullable 
	/// --inherited-- 
	pub created:DateTime<UTC>,
	/// @Value(users.user_id)
	/// --inherited-- 
	pub createdby:Option<Uuid>,
	/// @DisplayLength(100) When building a UI for this field
	/// @MaxLength(200) Do not go over 200 character on this one
	/// --inherited-- 
	pub description:Option<String>,
	/// --inherited-- 
	pub help:Option<String>,
	/// This is @Required it has @DisplayLength(50) - 50 character in display length a @MinLength(1) and @MaxLength(100) - Do not go over 100 characters or else the system will throw a ValueTooLong exception
	/// can also be express with @Length(1-100)
	/// --inherited-- 
	pub name:Option<String>,
	/// @Value(users.user_id) , which means the value will be set with the users.user_id value
	/// 
	/// @Where(users.active=true)
	/// --inherited-- 
	pub organization_id:Option<Uuid>,
	/// --inherited-- 
	pub priority:Option<f64>,
	/// defaults to: now()
	/// not nullable 
	/// --inherited-- 
	pub updated:DateTime<UTC>,
	/// @Value(users.user_id)
	/// --inherited-- 
	pub updatedby:Option<Uuid>,
	/// has one
	pub owner_id_users:Option<Users>,
	/// has one
	pub currency_id_currency:Option<Currency>,
	/// has one, extension table
	pub product_availability:Option<Box<ProductAvailability>>,
	/// has many, indirect referring table, derived from linker table: product_category
	pub category:Option<Vec<Category>>,
	/// has many, indirect referring table, derived from linker table: product_photo
	pub photo:Option<Vec<Photo>>,
	/// has many, indirect referring table, derived from linker table: product_review
	pub review:Option<Vec<Review>>,
}
```

```

##Roadmap
*become a full blown ORM
