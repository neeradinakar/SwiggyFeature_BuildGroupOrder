# Swiggy Together

A mobile-friendly Softr web application for coordinating shared restaurant orders, managing a group cart, and tracking individual bill shares.

> This application is a group-order coordination and bill-splitting tool. It does not submit orders to Swiggy and does not verify or process UPI payments unless a separate integration is configured.

## Purpose

Swiggy Together lets authenticated friends:

- Browse restaurants and menus connected to Airtable
- Create a group order for one restaurant
- Join an open group with a shareable code
- Add menu items to a shared cart
- Track separate item and bill totals
- Mark only their own payment share as paid

## Data Source

Use only the connected Airtable base **Swiggy Group Ordering** and the following six tables:

- `Users`
- `Restaurants`
- `Menu_Items`
- `Group_Orders`
- `Order_Items`
- `Bill_Splits`

Do not create replacement Softr databases, sample records, mock records, or extra tables. Missing fields or tables must be reported rather than silently substituted.

## Airtable Setup

The supplied CSV files are header-only templates. Import each CSV into its matching Airtable table, then configure field types and linked relationships.

### Relationships

Convert these foreign-key fields into Airtable linked-record fields:

| Source field | Linked table |
|---|---|
| `Menu_Items.Restaurant_ID` | `Restaurants` |
| `Group_Orders.Host_User_ID` | `Users` |
| `Group_Orders.Restaurant_ID` | `Restaurants` |
| `Order_Items.Group_Order_ID` | `Group_Orders` |
| `Order_Items.User_ID` | `Users` |
| `Order_Items.Menu_Item_ID` | `Menu_Items` |
| `Bill_Splits.Group_Order_ID` | `Group_Orders` |
| `Bill_Splits.User_ID` | `Users` |

Display linked-record names in Softr rather than raw IDs.

### Recommended Field Types

- Email: `Users.Email`
- Phone: `Users.Phone`
- URL: image and profile-image fields
- Checkbox: `Is_Active`, `Is_Veg`, and `Is_Available`
- Number/Currency: ratings, quantities, and INR amounts
- Date/Time: created, deadline, lock, and paid timestamps
- Single select: status, category, payment, and payment method fields

### Allowed Status Values

- `Group_Orders.Status`: `Open`, `Locked`, `Placed`, `Preparing`, `Delivering`, `Delivered`, `Cancelled`
- `Order_Items.Item_Status`: `Active`, `Removed`
- `Bill_Splits.Payment_Status`: `Pending`, `Paid`, `Failed`, `Waived`
- `Bill_Splits.Payment_Method`: `UPI`, `Cash`, `Other`
- `Bill_Splits.Split_Calculation_Method`: `Equal`, `ByItems`, `Custom`

Set monetary fields to INR where Airtable supports it. Keep numeric values available for calculations.

## Table Schemas

The templates provide the following fields:

### Users

`User_ID`, `Full_Name`, `Email`, `Phone`, `Profile_Image_URL`, `Default_Delivery_Address`, `Created_At`, `Is_Active`

### Restaurants

`Restaurant_ID`, `Restaurant_Name`, `Cuisine`, `Rating`, `Delivery_Time_Mins`, `Address`, `Image_URL`, `Is_Active`

### Menu_Items

`Menu_Item_ID`, `Restaurant_ID`, `Item_Name`, `Description`, `Category`, `Price_INR`, `Is_Veg`, `Is_Available`, `Image_URL`

### Group_Orders

`Group_Order_ID`, `Group_Name`, `Host_User_ID`, `Restaurant_ID`, `Shareable_Code`, `Status`, `Delivery_Address`, `Created_At`, `Order_Deadline`, `Locked_At`, `Subtotal_INR`, `Delivery_Fee_INR`, `Tax_INR`, `Total_Amount_INR`, `Notes`

### Order_Items

`Order_Item_ID`, `Group_Order_ID`, `User_ID`, `Menu_Item_ID`, `Quantity`, `Unit_Price_INR`, `Subtotal_INR`, `Item_Status`, `Special_Instructions`, `Created_At`

### Bill_Splits

`Bill_Split_ID`, `Group_Order_ID`, `User_ID`, `Individual_Share_INR`, `Payment_Status`, `Payment_Method`, `Paid_At`, `UPI_Reference`, `Split_Calculation_Method`, `Created_At`

## Navigation and Routes

| Page | Route | Access |
|---|---|---|
| Home | `/` | Public |
| Restaurants | `/restaurants` | Public |
| Restaurant Menu | `/restaurant-menu` | Sign-in required for ordering |
| Active Group Orders | `/group-orders` | Authenticated |
| Group Order Details | `/group-order-details` | Authenticated |
| My Split Bills | `/my-splits` | Authenticated |

### Header

- Home
- Restaurants
- Active Group Orders
- My Split Bills
- Sign In for logged-out users
- Profile and Sign Out for authenticated users

## Authentication and Access Control

- Map each Softr user to `Users` using email, preferably a normalized unique email field.
- Do not trust a user-selected `User_ID` as proof of identity.
- Redirect unauthenticated visitors to sign-in and return them to the requested page where supported.
- Enforce record-level visibility and update permissions with Airtable/Softr permissions in addition to hiding controls.
- Members can edit or remove only their own order items while the group is `Open`.
- Only the host can lock or manage the group order and change group-level settings/status.
- Members can mark only their own bill split as paid.
- Never expose private delivery addresses or individual bill data to public visitors.

## Pages

### Home

- Heading: `Order Together, Share the Feast`
- Supporting copy: `Start a group meal session, let friends add their favorite dishes, and split the bill seamlessly.`
- Primary action: `Browse Restaurants` to `/restaurants`
- Secondary action: `Join Group Order` to `/group-orders`
- Three-step explainer: Create a group, invite friends, split the bill.
- Show a sign-in prompt when a logged-out visitor tries to start or join a group.

### Restaurants Directory

- Source: `Restaurants`
- Display: restaurant name, cuisine, rating, delivery time, address, and image where available.
- Search by restaurant name and filter by cuisine.
- Filter out inactive restaurants when `Is_Active` exists and is configured.
- Format delivery time as `35 mins`.
- Include loading, empty, and no-results states.
- Opening a restaurant passes its record context to the menu page.

### Restaurant Menu

- Source the menu from `Menu_Items` linked to the selected `Restaurants` record.
- Show only available menu items where `Is_Available` is true.
- Display item name, description, category, price, and image.
- Show a green Veg badge or red Non-Veg badge.
- Format prices as INR, for example `₹250.00`.
- The add-to-cart form requires:
  - An eligible open group order owned or joined by the current authenticated user
  - The same restaurant as the selected menu item
  - Quantity with a minimum of `1`
  - Optional special instructions
- Set `User_ID`, `Menu_Item_ID`, and `Unit_Price_INR` from trusted server/Softr context rather than editable user input.
- Calculate `Subtotal_INR = Quantity × Unit_Price_INR`.
- If no eligible group exists, offer a create-group action for the selected restaurant.

### Active Group Orders

- Source: `Group_Orders`
- Default status filter: `Open`, `Locked`, `Placed`, `Preparing`, or `Delivering`
- History filters include `Delivered` and `Cancelled`.
- Display group name, restaurant, status, shareable code, total, and deadline.
- Create a new group with:
  - Group name
  - Restaurant
  - Delivery address
  - Optional deadline
  - Optional notes
- Set the host from the authenticated user and default status to `Open`.
- Generate a unique shareable code. Set created timestamps automatically where supported.
- A join-by-code flow should find an `Open` group and confirm the group and restaurant before joining.

### Group Order Details

#### Group Header

Display:

- Group name and host
- Restaurant
- Status and shareable code
- Delivery address for members only
- Deadline and total
- Locked timestamp where relevant

Provide copy/share controls for the invite code or link. Only the host may lock the group, confirm the action, and change group settings/status.

#### Shared Cart

Source `Order_Items` and filter them to the selected group order. Display item name, added-by user, quantity, unit price, subtotal, status, and special instructions where applicable.

- Editing/removal is allowed only for the authenticated user's own active items while status is `Open`.
- Confirm before removing an item.
- Once status is `Locked` or later, prevent additions and edits.
- Show clear empty-cart and locked-cart states.

#### Totals and Bill Splits

When fields are populated, display item subtotal, delivery fee, tax, and final total separately. Use a documented split method:

- `ByItems`: allocate each user's item subtotal, then allocate delivery fee and tax using a documented rule
- `Equal`: divide the relevant total evenly among participants
- `Custom`: use an explicit configured share

Create or update one bill-split record per participant for each recalculation. Do not create duplicates.

The `Mark My Share Paid` action must:

1. Update only the authenticated user's linked split record
2. Ask for confirmation
3. Record `Paid_At` where supported
4. Label the result as user-reported when no verified payment integration exists

### My Split Bills

- Source: `Bill_Splits`
- Filter records to the authenticated user's linked `Users` record.
- Default filter: `Payment_Status = Pending`
- Provide filters for paid and historical shares.
- Display group name, restaurant name, individual share, payment status, payment method, and date.
- A configured UPI link may open through `Pay via UPI`, but opening it must not mark the share paid.
- Without a verified integration, provide separate `Mark as paid` confirmation.

## Visual Design

- Brand: `Swiggy Together`
- Primary orange: `#FC8019`
- Dark slate text: `#282C3F`
- Page background: `#F4F4F5`
- White card surfaces
- Approximately 12px card radius
- Subtle shadows and generous spacing
- Accessible text and control contrast
- Clear loading, empty, success, error, and confirmation states
- Mobile-first controls and responsive tables/cards
- Thumb-friendly action sizing
- Sticky cart summary where practical
- Use real Airtable image URLs only

## Accessibility and Responsive Testing

- Use visible labels and keyboard-accessible controls.
- Maintain visible focus states.
- Use accessible modal and confirmation behavior.
- Preserve readable pricing, status, and table information on mobile.
- Test at desktop, tablet, and mobile widths.

## Data and Workflow Rules

1. Every group order belongs to exactly one restaurant.
2. Only authenticated users can create, join, add items, or view cart and split records.
3. Group items are editable only by their creator while the order is `Open`.
4. The host is responsible for locking and managing the group.
5. Statuses at or beyond `Locked` block cart changes.
6. Money is stored as a numeric value and displayed in INR.
7. Airtable is always the source of truth.
8. No API keys, secrets, sample records, or fake images belong in the frontend.

## Manual Implementation Checklist

The following items may require setup outside Softr’s native UI:

- [ ] Import each CSV into its matching Airtable table
- [ ] Confirm all six Airtable tables exist with the supplied fields
- [ ] Convert foreign-key fields into linked-record fields
- [ ] Configure recommended field types and allowed status values
- [ ] Connect the Airtable base named `Swiggy Group Ordering`
- [ ] Configure Softr authentication and map email to `Users.Email`
- [ ] Configure record-level visibility for groups, carts, addresses, and bill splits
- [ ] Configure host-only group locking and status controls
- [ ] Configure member-only editing of their own order items
- [ ] Create a unique/shareable-code workflow
- [ ] Configure Airtable automation for calculated totals and duplicate-safe bill splits
- [ ] Configure equal or item-based split allocation rules transparently
- [ ] Configure a UPI/payment link if payment-linking is required
- [ ] Configure verified payment integration if shares should change to `Paid` automatically
- [ ] Configure `Paid_At` updates and confirmation messaging
- [ ] Test desktop, tablet, and mobile layouts
- [ ] Verify unauthenticated visitors cannot access protected data or actions

## Known Platform Limitations

A manual Airtable automation or external integration is required when Softr cannot safely:

- Calculate cart totals automatically
- Recalculate and update one split per member without duplicates
- Generate a unique shareable code
- Record verified payment completion
- Enforce permissions beyond Softr/Airtable's supported record rules

Where a native action is unavailable, use the closest supported Airtable or Softr workflow and document the limitation in the implementation rather than pretending the operation exists.
