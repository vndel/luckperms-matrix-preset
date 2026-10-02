# luckperms-matrix-preset

![LuckPerms](https://img.shields.io/badge/LuckPerms-enterprise_preset-2D9CDB) ![Contexts](https://img.shields.io/badge/contexts-per_server%2Fworld-success) ![License](https://img.shields.io/badge/License-MIT-green)

> Permission hierarchy for a multi-server network: separated tracks, banded
> weights, and contextual nodes.

## Donor and staff are orthogonal axes

The central structural decision:

```yaml
tracks:
  donor:  [vip, vip-plus, mvp, mvp-plus, elite]
  staff:  [trial-helper, helper, moderator, senior-moderator, administrator]
  veterancy: [newcomer, regular, veteran, legend]
```

Three separate tracks, because they are independent dimensions. A moderator who
buys MVP must not lose moderator powers; a donor promoted to helper must keep
their donor perks. Modelling these as one ladder forces exactly those bugs, and
they are discovered in production when someone's rank disappears.

## Weight bands

| Band | Range | Purpose |
|---|---|---|
| Veterancy | 10-60 | Playtime-driven |
| Donor | 100-300 | Purchasable |
| Staff | 1000-2000 | Earned |

Weight decides prefix display order **and** which value wins when two groups
define the same meta key. Keeping the bands separated by an order of magnitude
means a staff prefix always outranks a donor prefix with no special-case logic.

## Contexts: one database, many servers

Context is what makes a single LuckPerms install workable across a network.
Without it you need a separate install per server, and a player's rank stops
following them.

```yaml
lobby-flight:
  permission: 'essentials.fly'
  value: true
  context:
    server: ['lobby-01', 'lobby-02', 'creative-01']

survival-no-flight:
  permission: 'essentials.fly'
  value: false
  context:
    server: 'survival-01'
```

Flight is fine in a lobby and ruinous in survival PvP. More specific context
wins, so the survival denial overrides the broader grant.

Two patterns worth copying:

**Trial staff are context-limited.** Powers apply only on the server where they
are supervised, not network-wide:

```yaml
trial-helper:
  contexts:
    server: ['survival-01']
```

**Arena disables bypass for everyone, including staff:**

```yaml
arena-no-bypass:
  permission: 'celestial.bypass'
  value: false
  context:
    world: 'pvp_arena'
```

A moderator who can walk out of an arena fight is a balance problem, not a
convenience.

## Meta over nodes for numeric limits

```yaml
mvp:
  meta:
    homes: '12'
    max-plots: '4'
    sell-multiplier: '1.35'
```

Limits belong in meta, not in `essentials.sethome.multiple.12`-style nodes.
Meta resolves by weight, is queryable by other plugins, and changing a number
does not mean editing a permission node in five groups.

## Files

| File | Contents |
|---|---|
| `config/tracks.yml` | Three promotion paths |
| `config/groups.yml` | Group definitions, inheritance, weights, meta |
| `config/contexts.yml` | Per-server and per-world conditional nodes |

## Applying

These are reference structures. Translate to commands, or adapt for
`lp export`:

```bash
lp creategroup vip
lp group vip parent add default
lp group vip meta set homes 3
lp group vip permission set essentials.kit.vip true server=survival-01
```

Verify before trusting:

```bash
lp user <name> info
lp user <name> permission check essentials.fly server=survival-01
```

## License

MIT — see [LICENSE](LICENSE).
