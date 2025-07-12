-- WARNING: This schema is for context only and is not meant to be run.
-- Table order and constraints may not be valid for execution.

CREATE TABLE public.budget (
  expense_category_id character varying NOT NULL,
  name character varying NOT NULL,
  category_type character varying NOT NULL CHECK (category_type::text = ANY (ARRAY['main'::character varying, 'sub'::character varying]::text[])),
  sub_category_name character varying,
  CONSTRAINT budget_pkey PRIMARY KEY (expense_category_id)
);
CREATE TABLE public.chart_of_accounts (
  account_sub1 character varying,
  account_code integer NOT NULL,
  account_name character varying NOT NULL UNIQUE,
  account_category character varying,
  account_type character varying NOT NULL CHECK (account_type::text = ANY (ARRAY['asset'::character varying, 'liability'::character varying, 'equity'::character varying, 'income'::character varying, 'expense'::character varying]::text[])),
  account_sub2 character varying,
  CONSTRAINT chart_of_accounts_pkey PRIMARY KEY (account_code)
);
CREATE TABLE public.contact (
  contact_id character varying NOT NULL,
  type text CHECK (type = ANY (ARRAY['Customer'::text, 'Supplier'::text, 'Other'::text])),
  name character varying NOT NULL,
  trade_license_name character varying,
  tacking_name character varying UNIQUE,
  address character varying,
  bin character varying,
  tin character varying,
  notes character varying,
  road_and_location character varying,
  city character varying,
  state character varying,
  country character varying,
  CONSTRAINT contact_pkey PRIMARY KEY (contact_id)
);
CREATE TABLE public.currency (
  currency_id character varying NOT NULL,
  CONSTRAINT currency_pkey PRIMARY KEY (currency_id)
);
CREATE TABLE public.department (
  department_id character varying NOT NULL,
  CONSTRAINT department_pkey PRIMARY KEY (department_id)
);
CREATE TABLE public.item (
  item_id integer NOT NULL,
  item_name character varying NOT NULL UNIQUE,
  item_sub_category character varying,
  item_category character varying,
  CONSTRAINT item_pkey PRIMARY KEY (item_id)
);
CREATE TABLE public.jnl (
  transaction_id character varying NOT NULL,
  transaction_line_id character varying NOT NULL,
  transaction_date date NOT NULL,
  description character varying NOT NULL,
  account_code integer NOT NULL,
  amount double precision NOT NULL,
  dc character varying NOT NULL CHECK (dc::text = ANY (ARRAY['debit'::character varying, 'credit'::character varying]::text[])),
  transaction_type character varying NOT NULL CHECK (transaction_type::text = ANY (ARRAY['bill'::character varying, 'invoice'::character varying, 'journal'::character varying, 'dnote'::character varying, 'cnote'::character varying, 'paid'::character varying, 'received'::character varying]::text[])),
  memo character varying NOT NULL,
  note character varying,
  contact_id character varying,
  status character varying NOT NULL CHECK (status::text = ANY (ARRAY['draft'::character varying, 'post'::character varying, 'void'::character varying]::text[])),
  currency_id character varying NOT NULL,
  total_amount double precision NOT NULL,
  due_date date,
  adjustment character varying,
  tracking character varying,
  created_date timestamp without time zone,
  related_transaction_id character varying,
  payment_transaction_id character varying,
  posting_period_id date,
  accounting_period_id date,
  created_by_id character varying,
  department_id character varying,
  location_id character varying,
  attachment_id character varying,
  item_id integer,
  quantity integer,
  rate double precision,
  expense_category_id character varying,
  vat_rate double precision CHECK (vat_rate >= 0::double precision AND vat_rate <= 100::double precision),
  vat_amount double precision,
  tax_rate double precision CHECK (tax_rate >= 0::double precision AND tax_rate <= 100::double precision),
  tax_amount double precision,
  tracking_name1 character varying,
  tracking_name2 character varying,
  tracking_name3 character varying,
  repet_post_start date,
  repet_post_frequency integer,
  repet_post_due integer,
  repet_post_end date,
  repet_post_type character varying CHECK (repet_post_type::text = ANY (ARRAY['draft'::character varying, 'post'::character varying]::text[])),
  repet_post_note character varying,
  reminder_start date,
  reminder_frequency integer,
  reminder_end date,
  CONSTRAINT jnl_pkey PRIMARY KEY (transaction_line_id),
  CONSTRAINT jnl_location_id_fkey FOREIGN KEY (location_id) REFERENCES public.location(location_id),
  CONSTRAINT jnl_created_by_id_fkey FOREIGN KEY (created_by_id) REFERENCES public.user(user_id),
  CONSTRAINT jnl_item_id_fkey FOREIGN KEY (item_id) REFERENCES public.item(item_id),
  CONSTRAINT jnl_currency_id_fkey FOREIGN KEY (currency_id) REFERENCES public.currency(currency_id),
  CONSTRAINT jnl_contact_id_fkey FOREIGN KEY (contact_id) REFERENCES public.contact(contact_id),
  CONSTRAINT jnl_expense_category_id_fkey FOREIGN KEY (expense_category_id) REFERENCES public.budget(expense_category_id),
  CONSTRAINT jnl_account_code_fkey FOREIGN KEY (account_code) REFERENCES public.chart_of_accounts(account_code),
  CONSTRAINT jnl_department_id_fkey FOREIGN KEY (department_id) REFERENCES public.department(department_id)
);
CREATE TABLE public.location (
  location_id character varying NOT NULL,
  CONSTRAINT location_pkey PRIMARY KEY (location_id)
);
CREATE TABLE public.user (
  user_id character varying NOT NULL,
  CONSTRAINT user_pkey PRIMARY KEY (user_id)
);
