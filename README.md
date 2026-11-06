# merchant-archetype

The **Merchant** archetype bundle for [protoAgent](https://github.com/protoLabsAI/protoAgent):
an inventory and selling assistant for anyone who buys in lots and sells piece by piece.
Scan a box in from your phone, split it into pieces, label and bin every one, price it with
the evidence kept, draft the listing, record the sale and see what each purchase actually made.

**It never posts or messages anyone.** It drafts listings and reads the sales you paste in,
and you list, sell and ship yourself. There is no marketplace write access in the bundle.

## What it does

| Phase | What happens | You see |
|---|---|---|
| **Bring a box in** | Photograph it and say what you paid. The agent identifies it and the pieces it splits into, and you choose Keep or Sell for each | A new lot and its pieces in the **Inventory** view |
| **Label and bin** | Print QR labels, stick them on, then put pieces away by scanning a piece's label and the bin's | Every piece in a bin, ready to find |
| **Price** | Targets set from comparable sales, each with the basis it rests on and the date | Prices you can trace, flagged when they go stale |
| **List** | A title and description drafted from the record and photos | Copy buttons for eBay; you post it and paste the link back |
| **Sell and pack** | Paste an order or a quick note and the agent reads it; you confirm | The sale recorded, the piece on the To pack list with its bin, and a reminder to end the listing anywhere else |
| **Review** | Profit per lot, aging stock, sealed vs split, a weekly review if you want it | What's selling, what's sitting, and what to do about it |

## The rules it keeps

The persona is protoAgent's `merchant` soul preset:

1. **Never posts, lists, reprices live, or contacts a buyer.** It prepares; you act.
2. **The inventory is the source of truth.** A number that isn't recorded hasn't been decided.
3. **A price always has a basis.** Sold comps, asking prices or retail, said plainly, with a date.
   Asking prices and retail are never passed off as what buyers paid.
4. **Never invents facts about an item.** What it can't see or confirm stays unknown.
5. **Sold means a recorded sale**, with its price, fees and postage, never a flipped status.

## What's inside

| plugin | source | role |
|---|---|---|
| `inventory` | [inventory-plugin](https://github.com/protoLabsAI/inventory-plugin), floor `v0.16.2` | Lots and pieces, bins and QR labels, phone scan intake, listing drafts, paste-a-sale, packing, aging stock, lot P&L, and the **Inventory** view |
| `artifact` | builtin | Side-by-side price and profit comparisons |
| `notes` | builtin | Decisions and loose ends between sessions |

The bundle explicitly enables all three: `inventory` is disabled by author default. The persona
lives in core as the `merchant` preset; the picker catalog row also lives in core and supplies
the id `merchant`.

Two optional setup questions: your timezone (for the weekly review's schedule) and whether to
turn the weekly review on. Everything else uses the plugin's defaults.

**Not included: eBay sold comps.** The separate [ebay plugin](https://github.com/protoLabsAI/ebay-plugin)
reads eBay's sold listings through a browser session you sign in to. It isn't in protoAgent's
catalog, because automated access to eBay is restricted by eBay's user agreement, so this bundle
leaves it out. Without it, the agent prices from what it can research on the web and what you
confirm, and labels the basis honestly.

## Create an agent

**Core floor: protoAgent ≥ 0.148.0.** Inventory declares `min_protoagent_version: 0.148.0`; the
loader refuses it on older cores. Printed QR labels open their item or bin in the console on
protoAgent ≥ 0.227.0. Scanning a box in needs a vision-capable model. The bundle targets
`0.229.0` for verification. Bundles cannot enforce a core floor themselves.

The `merchant` soul preset ships with the protoAgent PR adding the catalog row. On older cores
without that preset, `soul_preset` falls back to the base persona.

Merchant is **held** from the new-agent picker until it has been tested end to end.

### From the picker (once the archetype is listed)

Choose **Fleet ▸ New agent ▸ Merchant ▸ Create** once the core catalog lists it.

Until then, use **Settings ▸ Plugins ▸ Install from URL** →
`https://github.com/protoLabsAI/merchant-archetype`. An installed bundle with an `archetype:`
block registers a picker card and installs its members on the hub. If the host lacks the
preset, paste the persona into **Advanced ▸ Persona** during setup.

### From the API

Fetch the core persona and send it inline, so the host need not already ship the preset:

```bash
curl -fsSL https://raw.githubusercontent.com/protoLabsAI/protoAgent/main/config/soul-presets/merchant.md -o merchant.md

jq -n --rawfile soul merchant.md '{
  name: "merchant",
  bundle: "https://github.com/protoLabsAI/merchant-archetype",
  soul: $soul,
  requires_tools: ["inventory_summary", "inventory_upsert_item", "inventory_set_price", "inventory_mark_sold"]
}' | curl -s -X POST http://127.0.0.1:7870/api/fleet \
      -H 'content-type: application/json' \
      ${PROTOAGENT_TOKEN:+-H "authorization: Bearer $PROTOAGENT_TOKEN"} \
      -d @-
```

`7870` is the default instance port; use your hub's port. The member inherits the hub's model
connection.

### From the CLI

```sh
python -m server workspace new shop --bundle https://github.com/protoLabsAI/merchant-archetype \
  --soul merchant.md --input inventory.review_timezone=America/Los_Angeles
```

Without `--soul` the workspace has no persona, and the CLI never records the capability
contract; the picker and `POST /api/fleet` do both.

## Maintaining the pins

`.github/workflows/verify-bundle.yml` installs this manifest's pin set into a scratch agent on a
fresh protoAgent checkout, loads every member, probes every declared view and checks that every
tool in `requires_tools` binds. It runs on every pull request, weekly, and when a member releases.
The weekly job opens or updates one `bump-pins` PR when a member has a newer release; that PR
has to pass `verify` before it merges.

## License

MIT, see [LICENSE](LICENSE).
