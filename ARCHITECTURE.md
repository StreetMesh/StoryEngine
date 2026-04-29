# Story Engine

**A Story Engine for the spatial web.**
An open-source Laravel package that gives Places in StreetMesh a living narrative — driven by a configurable, persistent, rules-aware ensemble of AI agents. Equally suitable for **gaming**, **training**, **commerce**, **events**, and **community**.

> Repo: `StreetMesh/StoryEngine`
> Composer: `composer require streetmesh/story-engine`
> Namespace: `StreetMesh\StoryEngine`
> License: **MIT** (code) — aligned with sibling StreetMesh packages like [`streetmesh/avatars`](https://github.com/StreetMesh/Avatars)

---

## 0. The Dream, in One Paragraph

> *We dream of a virtual world where people can live active, productive lives full of wonder and community. A place that is spatialized, decentralized, interoperable, and humane. Stores, domiciles, classrooms, gathering places — all sharing one Street.*

A Place without a story is an empty room. The **Story Engine** is the thing that turns Places into *somewheres*: it animates NPCs, runs games and training, hosts events, narrates discoveries, and remembers what happened. It is to a StreetMesh Server what `streetmesh/avatars` is to identity — a foundational package that any Server can install to make its Places feel alive.

---

## 1. Filtering Through The Dream — Design Constraints

Every architectural decision below is filtered through StreetMesh's stated values and ontology. The constraints are not aesthetic — they shape the API.

### Spatialized
A story is **never floating in the void**. Every Scene is anchored to a Place (a StreetMesh location with coordinates within a Server's spatial graph). The Director knows *where* it is and can describe it.

### Decentralized
The Story Engine runs **inside a single StreetMesh Server**. It does not depend on a central matchmaker, story registry, or shared LLM gateway. Two Servers running Story Engine are sovereign peers — they may federate stories via the Protocol, but neither requires the other.

### Interoperable
- **Avatars** are first-class citizens, not custom Character models. NPCs are Program Avatars from `streetmesh/avatars`. PCs are Person Avatars. Places are Place Avatars. Items are Thing Avatars.
- **Lore** is queryable via standard StreetMesh content discovery (`MeshObject` microdata, `StreetMaps` indexing) — not a private opaque table.
- **Events** emitted by the engine are broadcastable over the StreetMesh Protocol so neighboring Places, multiplayer Hubs, and external clients can react.

### Humane
The package ships with an explicit **safety preamble** in every Director prompt: no harm to real persons, no manipulation, no romantic/sexual content with minors, no weaponization of personal data. Operators can extend the preamble; they cannot remove the floor.

### Modular
Three pluggable seams:
1. **Ruleset** — the mechanics layer (dice, rubrics, none).
2. **Content Pack** — the genre/domain bundle (fantasy game, training scenario, retail concierge, event host).
3. **Locale Adapter** — how the engine binds to a specific Place geometry, lore, and Avatars on a Server.

### Accessible
Narration is generated as **structured beats** — text first, with optional audio/voice/visual layers. Screen-reader friendly by default. The same beat that drives a 3D experience drives a 2D fallback.

### Creative
The package is designed so that **a Place owner can author a Story Engine experience without writing PHP**: a Content Pack + a few Eloquent rows in seed data is enough to stand up a working scene. PHP is for *new genres*, not new stories.

### Transparent
Every prompt, every tool call, every state mutation is observable through Laravel events and a `story-engine:trace` command. Nothing the Director does is hidden from the Server operator.

---

## 2. Vocabulary (Reframed Through StreetMesh)

| StreetMesh term | Story Engine binding |
|---|---|
| **Server** | Hosts the engine; owns Worlds and Campaigns. |
| **Place** | A spatial location on the Server. A Scene runs *at* a Place. |
| **Place Avatar** | The "skin" of a Place. The Director may describe and modify atmospheric/cosmetic aspects within ruleset bounds. |
| **Person Avatar** | A real user. Becomes a **Player** when they enter a Story. |
| **Program Avatar** | An NPC. Driven by the Director through dialogue/action tools. |
| **Thing Avatar** | An item. Inventory, props, set dressing. |
| **MeshObject** | Schema.org-style microdata describing 3D objects and lore — used as a discovery surface for `RecallLore`. |
| **StreetMap** | Site-level index. The engine publishes Story-discoverable surfaces here so neighbors can find them. |
| **Hub** (Colyseus) | Real-time multiplayer authority. The engine publishes beats to a Hub room when a Story has multiple synchronous Players. |
| **Portal/Window** | Real-world video bridges. Out of scope for v1, but the beat schema preserves a `media` channel so a future Director can narrate a livestreamed event. |

| Story Engine term | What it is |
|---|---|
| **World** | A configured narrative substrate (lore + ruleset + content pack). |
| **Campaign** | A play-through of a World by a Party at a Place. |
| **Scene** | The current dramatic unit, anchored to a Place. |
| **Beat** | A single atomic event (narration, dialogue, check, choice, outcome). The canonical unit of story state. |
| **Director** | The DM agent. Voices NPCs, narrates, calls tools. |
| **Choreographer** | Plans Scene structure ahead of the Director. |
| **Chronicler** | Compresses history into long-term memory. |
| **Concierge** *(optional)* | A non-narrative agent variant for commerce/wayfinding/event-hosting Places. Same engine, restricted toolset. |

---

## 3. The Three Modes (and why "Dungeon Master" is too narrow)

Inside The Dream, the Story Engine animates more than dungeons. The same core supports three operational modes, configured per Campaign:

### 3.1 Game Mode
Classic DM experience. Director with full toolset (dice, checks, NPC dialogue, scene transitions, branching choices). Suits tabletop-style adventures, escape rooms, mystery experiences.

### 3.2 Training Mode
Director runs a **Rubric Ruleset** — no dice, every Player utterance is scored against an explicit competency rubric. Suits role-play training, sales practice, soft-skill assessment, language immersion. The Director's job becomes "be a believable counterparty *and* generate a defensible score."

### 3.3 Concierge Mode
Director persona is hospitable, not adversarial. Toolset is restricted to lookup-only (no checks, no choice gating). Suits storefronts, info kiosks, event venues, museum guides. The Director tells the Place's story to a visitor and helps them find what they came for.

A single Server can run all three — each Place chooses its mode.

---

## 4. High-Level Architecture

```
┌────────────────────────────────────────────────────────────────────────┐
│                       StreetMesh Server (Laravel)                       │
│                                                                         │
│   streetmesh/avatars   streetmesh/story-engine   streetmesh/streettiles │
│         │                       │                        │              │
│         ▼                       ▼                        ▼              │
│   ┌──────────────────────────────────────────────────────────────────┐  │
│   │                       Server Services                            │  │
│   │   AvatarRegistry   StoryEngine   PlaceGraph   MeshObjectIndex   │  │
│   └────────────────────────────────┬─────────────────────────────────┘  │
└────────────────────────────────────┼────────────────────────────────────┘
                                     │
              ┌──────────────────────┼──────────────────────┐
              ▼                      ▼                      ▼
   ┌────────────────────┐ ┌──────────────────┐ ┌────────────────────┐
   │  StreetMesh        │ │  Web Browser     │ │  External clients  │
   │  Browser (XR/3D)   │ │  (2D fallback)   │ │  via Protocol      │
   └────────────────────┘ └──────────────────┘ └────────────────────┘

         (Inside Story Engine)

   ┌────────────────────────────────────────────────────────────────┐
   │                    StoryEngine Facade                          │
   │     ->place($id)->campaign($id)->advance($playerInput)         │
   └────────────────────────────────┬───────────────────────────────┘
                                    │
            ┌───────────────────────┼────────────────────┐
            ▼                       ▼                    ▼
   ┌────────────────┐    ┌──────────────────┐   ┌──────────────────┐
   │  Director      │    │  Choreographer   │   │  Chronicler      │
   │  (DM agent)    │    │  (planning)      │   │  (memory)        │
   └────────┬───────┘    └────────┬─────────┘   └────────┬─────────┘
            │                     │                       │
            └─────────────────────┼───────────────────────┘
                                  ▼
   ┌────────────────────────────────────────────────────────────────┐
   │                       Tool Layer                               │
   │  RollDice  ResolveCheck  RecallLore  DescribePlace             │
   │  GetAvatar  ListInventory  ProposeBeat  RequestPlayerChoice    │
   │  UpdateAvatarState  PublishProtocolEvent  EndScene             │
   └─────────────────────────────┬──────────────────────────────────┘
                                 ▼
   ┌────────────────────────────────────────────────────────────────┐
   │                State Layer (Eloquent + Actions)                 │
   │  World  Campaign  Scene  Beat  Party  PlaceBinding  ...        │
   │            (NPCs/PCs/Items reference Avatar IDs)                │
   └─────────────────────────────┬──────────────────────────────────┘
                                 ▼
   ┌────────────────────────────────────────────────────────────────┐
   │      Ruleset (Generic | D20 | Rubric | custom)                  │
   └────────────────────────────────────────────────────────────────┘
                                 │
                                 ▼
   ┌────────────────────────────────────────────────────────────────┐
   │   Outbound: BeatGenerated → SSE / Reverb / Hub room / Protocol │
   └────────────────────────────────────────────────────────────────┘
```

### Three-agent ensemble (refined)

1. **Choreographer** — runs at scene boundaries. Cheap model. Structured output. Produces a `ScenePlan`.
2. **Director** — runs during a beat. Stronger model. Streams. Calls tools. Voices NPCs.
3. **Chronicler** — runs every N beats and at scene end. Cheap model. Structured output. Compresses history; updates relationships and lore.

This pattern matches Anthropic's documented multi-agent compositions (orchestrator + workers, prompt chaining) and the Laravel AI SDK's first-class support for both.

---

## 5. Package Layout

```
streetmesh/story-engine/
├── composer.json
├── LICENSE                          # MIT
├── README.md
├── DESIGN.md                        # Mirrors Avatars convention
├── config/
│   └── story-engine.php
├── database/
│   └── migrations/
│       ├── create_story_worlds_table.php
│       ├── create_story_campaigns_table.php
│       ├── create_story_place_bindings_table.php
│       ├── create_story_scenes_table.php
│       ├── create_story_beats_table.php
│       ├── create_story_parties_table.php
│       ├── create_story_party_members_table.php
│       ├── create_story_avatar_roles_table.php       # NPC/PC/Item bindings
│       ├── create_story_factions_table.php
│       ├── create_story_relationships_table.php
│       ├── create_story_lore_entries_table.php
│       └── create_story_dice_rolls_table.php
├── routes/
│   └── api.php                      # Mirrors Avatars: configurable prefix + middleware
├── resources/
│   ├── views/                       # Optional Blade for admin/debug + Filament-friendly
│   └── stubs/                       # Stubs for make:story-pack
├── packages/                        # Frontend, mirroring Avatars layout
│   ├── story-vue/                   # @streetmesh/story-vue
│   ├── story-react/                 # @streetmesh/story-react
│   └── story-alpine/                # @streetmesh/story-alpine
├── src/
│   ├── StoryEngineServiceProvider.php
│   ├── Facades/
│   │   └── Story.php                # Avatars uses `Avatar` — we use `Story`
│   ├── StoryEngine.php
│   │
│   ├── Agents/
│   │   ├── DirectorAgent.php
│   │   ├── ChoreographerAgent.php
│   │   ├── ChroniclerAgent.php
│   │   └── ConciergeAgent.php       # Restricted Director variant
│   │
│   ├── Tools/
│   │   ├── RollDice.php
│   │   ├── ResolveCheck.php
│   │   ├── RecallLore.php           # Vector / SimilaritySearch
│   │   ├── DescribePlace.php        # Reads MeshObject + Place data
│   │   ├── GetAvatar.php            # Reads from streetmesh/avatars
│   │   ├── ListInventory.php
│   │   ├── ProposeBeat.php
│   │   ├── RequestPlayerChoice.php
│   │   ├── UpdateAvatarState.php    # Validated; ruleset-allowlisted
│   │   ├── PublishProtocolEvent.php # Emits to neighboring Servers/Hubs
│   │   └── EndScene.php
│   │
│   ├── Contracts/
│   │   ├── Ruleset.php
│   │   ├── DiceRoller.php
│   │   ├── CheckResolver.php
│   │   ├── ScoreRubric.php
│   │   ├── ContentPack.php
│   │   └── PlaceBinder.php          # Adapter: how a Campaign attaches to a Place
│   │
│   ├── Rulesets/
│   │   ├── GenericNarrativeRuleset.php
│   │   ├── D20Ruleset.php
│   │   └── RubricRuleset.php
│   │
│   ├── Modes/
│   │   ├── GameMode.php
│   │   ├── TrainingMode.php
│   │   └── ConciergeMode.php
│   │
│   ├── Models/
│   │   ├── World.php
│   │   ├── Campaign.php
│   │   ├── PlaceBinding.php
│   │   ├── Scene.php
│   │   ├── Beat.php
│   │   ├── Party.php
│   │   ├── PartyMember.php
│   │   ├── AvatarRole.php           # NPC/PC/Item role at scope of Campaign
│   │   ├── Faction.php
│   │   ├── Relationship.php
│   │   ├── LoreEntry.php
│   │   └── DiceRoll.php
│   │
│   ├── Actions/
│   │   ├── StartCampaign.php
│   │   ├── AdvanceBeat.php
│   │   ├── ApplyAvatarStatePatch.php
│   │   ├── GrantThingToAvatar.php
│   │   ├── ChangeRelationship.php
│   │   ├── SummarizeHistory.php
│   │   └── PublishToProtocol.php
│   │
│   ├── Schemas/
│   │   ├── BeatSchema.php
│   │   ├── ScenePlanSchema.php
│   │   ├── HistorySummarySchema.php
│   │   └── RubricScoreSchema.php
│   │
│   ├── Events/
│   │   ├── CampaignStarted.php
│   │   ├── CampaignAdvanced.php
│   │   ├── SceneStarted.php
│   │   ├── SceneEnded.php
│   │   ├── BeatGenerated.php        # Broadcastable
│   │   ├── DiceRolled.php
│   │   ├── PlayerChoiceRequested.php
│   │   ├── PlayerChoiceResolved.php
│   │   └── HistorySummarized.php
│   │
│   ├── Protocol/                    # StreetMesh Protocol bridges
│   │   ├── BeatPublisher.php        # Publishes BeatGenerated as Protocol events
│   │   ├── HubBridge.php            # Pushes beats into Colyseus Hub rooms
│   │   └── MeshObjectReader.php     # Reads schema.org microdata as lore source
│   │
│   ├── Support/
│   │   ├── PromptComposer.php
│   │   ├── ContextBudget.php
│   │   ├── SafetyPreamble.php
│   │   └── Streaming/
│   │       ├── ServerSentEventStream.php
│   │       └── BroadcastStream.php
│   │
│   └── Console/
│       ├── InstallCommand.php       # story-engine:install (mirrors avatars:install)
│       ├── PublishCommand.php       # story-engine:publish (mirrors avatars:publish)
│       ├── MakeContentPackCommand.php
│       ├── CampaignTickCommand.php
│       └── TraceCommand.php         # story-engine:trace {campaign}
│
└── tests/
    ├── Feature/
    └── Unit/
```

**Convention parity with `streetmesh/avatars`:**
- `php artisan story-engine:install` provisions config, migrations, storage, frontend deps.
- `php artisan story-engine:publish --config|--migrations|--all` mirrors `avatars:publish`.
- API routes register under a configurable prefix with `auth:sanctum` middleware by default.
- Frontend ships as `@streetmesh/story-vue`, `@streetmesh/story-react`, `@streetmesh/story-alpine` packages.

---

## 6. Data Model

### 6.1 Core tables

**`story_worlds`**
- `id`, `slug`, `name`, `description`, `content_pack` (string), `ruleset` (string), `mode` (`game|training|concierge`), `metadata` (json), timestamps

**`story_place_bindings`** *(the spatial anchor)*
- `id`, `world_id`, `place_id` (FK or URN to a StreetMesh Place), `place_metadata` (json — cached lat/lng/extent), `default_scene_template` (nullable), `discovery_visibility` (`private|server|federated`)
- Allows the same World to bind to multiple Places (e.g., a chain of training rooms).

**`story_campaigns`**
- `id`, `world_id`, `place_binding_id`, `owner_id` (polymorphic — Server's User or external identity), `name`, `status`, `current_scene_id`, `summary` (text), `metadata` (json), timestamps

**`story_scenes`**
- `id`, `campaign_id`, `place_id` (denormalized — a Scene can move within a Place graph), `title`, `setting` (text), `intent` (text), `status`, `started_at`, `ended_at`

**`story_beats`**
- `id`, `scene_id`, `sequence`, `actor_type` (`director|player|npc|system`), `actor_avatar_id` (nullable — references `streetmesh/avatars`), `kind` (`narration|dialogue|action|check|choice|outcome|protocol`), `content` (text), `payload` (json), `media` (json — optional audio/image/3D references), `created_at`

**`story_parties`** + **`story_party_members`**
- A Party groups one or more Person Avatars (Players) for a Campaign. `party_members` joins Person Avatars to a Party with optional role labels.

**`story_avatar_roles`** *(replaces standalone Character model)*
- `id`, `campaign_id`, `avatar_id` (FK to `streetmesh/avatars`), `role` (`pc|npc|item|set_dressing`), `archetype`, `stats` (json), `traits` (json), `goals` (text), `secrets` (text), `state` (json), `controllability` (`director|player|both`)
- **No new identity layer.** Every "character" is an Avatar that already exists in the Server's avatar registry. The Story Engine adds *roles and state*, never identity.

**`story_factions`**, **`story_relationships`**, **`story_lore_entries`**, **`story_dice_rolls`** — `relationships` references Avatar IDs.

### 6.2 Lore as a federated surface

`story_lore_entries` stores embeddings for fast RAG, but the canonical lore source is **MeshObject microdata** on the Server's existing pages. The `RecallLore` tool queries:
1. Local `story_lore_entries` (campaign-scoped) — fast path.
2. Server-local MeshObjects via `Protocol\MeshObjectReader` — Place-scoped fallback.
3. *(Future)* Federated neighbors via the StreetMesh Protocol — only if `discovery_visibility=federated`.

This keeps the engine consistent with The Dream's "lore lives on the web, not in a private database."

### 6.3 Conversation persistence

Director uses the AI SDK's `RemembersConversations` trait, keyed off `campaign_id`. The `agent_conversations` table is the LLM's working memory; `story_beats` is the canonical narrative record.

---

## 7. The Three Agents

### 7.1 DirectorAgent

```php
namespace StreetMesh\StoryEngine\Agents;

use Laravel\Ai\Attributes\{Model, Provider, Temperature, MaxSteps};
use Laravel\Ai\Concerns\RemembersConversations;
use Laravel\Ai\Contracts\{Agent, Conversational, HasTools};
use Laravel\Ai\Enums\Lab;
use Laravel\Ai\Promptable;
use StreetMesh\StoryEngine\Models\Campaign;
use StreetMesh\StoryEngine\Support\PromptComposer;
use Stringable;

#[Provider(Lab::Anthropic)]
#[Model('claude-sonnet-4-6')]
#[Temperature(0.85)]
#[MaxSteps(12)]
class DirectorAgent implements Agent, Conversational, HasTools
{
    use Promptable, RemembersConversations;

    public function __construct(public Campaign $campaign) {}

    public function instructions(): Stringable|string
    {
        return PromptComposer::director($this->campaign);
    }

    public function tools(): iterable
    {
        return $this->campaign->mode()->toolsFor($this->campaign);
    }
}
```

Mode determines the tool palette. **Concierge mode strips dice, checks, mutation, and protocol-publish** — leaving only lookup and narration. **Training mode adds rubric tools.** **Game mode includes everything.** This is the mechanism that makes one Director pluggable across radically different Places.

### 7.2 ChoreographerAgent — produces a `ScenePlan`

```json
{
  "title": "First contact at the Apothecary",
  "setting": "The shop is dim, smelling of camphor and old paper. Rain on the window.",
  "intent": "Establish trust, hint at the cult, surface the antidote question.",
  "place_id": "place_apothecary_main",
  "beats_minimum": 3,
  "beats_maximum": 7,
  "required_elements": ["a moral cost", "a clue about the cult"],
  "available_avatar_role_ids": [12, 17],
  "fail_state": "...",
  "success_state": "..."
}
```

Note `place_id` — every plan is spatially anchored.

### 7.3 ChroniclerAgent — produces a `HistorySummary`

```json
{
  "summary": "Compressed prose retelling.",
  "avatar_state_updates": [
    {"avatar_role_id": 7, "field": "state.mood", "value": "wary"}
  ],
  "relationship_changes": [
    {"from_avatar_id": 1, "to_avatar_id": 12, "delta": -15, "reason": "..."}
  ],
  "new_lore": [
    {"title": "...", "body": "...", "tags": ["cult","villain"]}
  ],
  "protocol_publishable": false
}
```

The `protocol_publishable` flag lets the Chronicler decide whether a summary is shareable with neighboring Servers.

---

## 8. Tools — Detail

Same Tool contract as the AI SDK: `description`, `schema`, `handle`. All tools scoped via constructor; **no IDs trusted from prompts.**

### Core tools (all modes)

- **`RecallLore`** — local + MeshObject + (optionally) federated lore.
- **`DescribePlace`** — reads Place + MeshObject data into a structured description; **does not invent geometry**.
- **`GetAvatar`** — reads from `streetmesh/avatars`. Returns role + visible state (not secrets).
- **`ProposeBeat`** — persists narration / dialogue / outcome.
- **`RequestPlayerChoice`** — yields control to the application; returns persisted `PlayerChoiceRequest`.

### Game-mode + Training-mode tools

- **`RollDice`** — delegates to Ruleset; logged in `story_dice_rolls` with seed for replay.
- **`ResolveCheck`** — Ruleset-driven outcome resolution.
- **`UpdateAvatarState`** — patch through `ApplyAvatarStatePatch` Action; allowlist enforced by Ruleset.
- **`ListInventory`** — Things owned by an Avatar in this Campaign.
- **`EndScene`** — marks scene resolved, queues Choreographer for the next.

### Spatial-web tools (opt-in across modes)

- **`PublishProtocolEvent`** — emits a Protocol-compliant event for neighboring Servers / Hub rooms / external clients. Subject to `discovery_visibility` on the PlaceBinding.

### Concierge-only restrictions

Concierge mode disables `RollDice`, `ResolveCheck`, `UpdateAvatarState`, and `RequestPlayerChoice` (a concierge doesn't gate visitors with branching choices — it answers questions and offers options).

---

## 9. Ruleset Plugin Contract

```php
namespace StreetMesh\StoryEngine\Contracts;

use StreetMesh\StoryEngine\Models\Campaign;

interface Ruleset
{
    public function name(): string;

    public function diceRoller(): DiceRoller;

    public function checkResolver(): CheckResolver;

    /** Avatar role state fields the Director may mutate. */
    public function mutableAvatarFields(): array;

    /** Mode-aware additional tools. */
    public function additionalTools(Campaign $campaign): array;

    /** Director system-prompt fragment describing rules. */
    public function directorBriefing(): string;

    /** Optional rubric (Training mode). */
    public function rubric(): ?ScoreRubric;
}
```

Reference rulesets:
1. **`GenericNarrativeRuleset`** — no dice; pure improv. Default.
2. **`D20Ruleset`** — `1d20 + modifier vs DC`, crit on 20/1.
3. **`RubricRuleset`** — every Player utterance is scored against a competency rubric; aggregate exposed via `Campaign::rubricScore()`.

---

## 10. Place Binder Contract

This is the seam unique to filtering through The Dream. A `PlaceBinder` adapts a Campaign to a specific Place geometry on the host Server.

```php
namespace StreetMesh\StoryEngine\Contracts;

interface PlaceBinder
{
    /** Resolve the spatial extent of this Place — used by DescribePlace. */
    public function extent(string $placeId): PlaceExtent;

    /** Avatars currently within or adjacent to this Place. */
    public function avatarsAt(string $placeId): iterable;

    /** Schema.org-style MeshObjects discoverable at this Place. */
    public function meshObjects(string $placeId): iterable;

    /** Optional: handoff coordinates when a Scene moves within a Place graph. */
    public function navigate(string $fromPlaceId, string $direction): ?string;
}
```

A Server bundles its own `PlaceBinder` implementation that knows how its spatial graph works. The Story Engine never assumes a specific tile format, geometry library, or coordinate system — it asks the Binder.

---

## 11. Public API

### 11.1 Facade-level usage

```php
use StreetMesh\StoryEngine\Facades\Story;

// Bind a World to a Place
$binding = Story::worlds()
    ->find('mistmoor-fantasy')
    ->bindToPlace('place_apothecary_main', visibility: 'server');

// Start a Campaign with a Party of Person Avatars
$campaign = $binding
    ->newCampaign(name: 'The Ashen Moth Affair')
    ->withParty([$aliceAvatar, $bobAvatar])     // Person Avatars from streetmesh/avatars
    ->start();

// Advance
$response = Story::campaign($campaign)->advance(
    PlayerInput::say($aliceAvatar, "I check the bookshelf for hidden compartments.")
);

// Streaming
foreach ($response->stream() as $chunk) { /* ... */ }
```

### 11.2 Three-mode usage examples

**Game mode** — a tavern mystery in a virtual pub Place:
```php
$world = Story::worlds()->create([
    'slug' => 'silver-stag-mystery',
    'content_pack' => 'fantasy-osr',
    'ruleset' => 'd20',
    'mode' => 'game',
]);
```

**Training mode** — customer-escalation practice in a virtual support office:
```php
$world = Story::worlds()->create([
    'slug' => 'enterprise-escalation',
    'content_pack' => 'training-scenario',
    'ruleset' => 'rubric',
    'mode' => 'training',
]);
```

**Concierge mode** — a virtual storefront welcoming a visitor:
```php
$world = Story::worlds()->create([
    'slug' => 'storefront-welcome',
    'content_pack' => 'retail-concierge',
    'ruleset' => 'generic',
    'mode' => 'concierge',
]);
```

The API surface is identical. Only the Mode and Content Pack differ.

### 11.3 Streaming via Reverb / SSE

The `BeatGenerated` event is broadcastable; pair with `streetmesh/story-{vue|react|alpine}` to render a chat-like beat stream into a 2D fallback UI, or feed it to a 3D StreetMesh Browser for spatialized audio/visual presentation.

### 11.4 REST API (mirrors Avatars conventions)

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/story/campaigns` | Start a Campaign |
| `GET` | `/api/story/campaigns/{id}` | Campaign status + recent beats |
| `POST` | `/api/story/campaigns/{id}/advance` | Submit Player input, get beats |
| `POST` | `/api/story/campaigns/{id}/choices/{choiceId}` | Resolve a pending choice |
| `GET` | `/api/story/campaigns/{id}/stream` | SSE stream of beats |
| `GET` | `/api/story/places/{placeId}/campaigns` | Campaigns active at a Place |

---

## 12. The Protocol Bridge

Story Engine state changes that affect the **shared spatial world** (an NPC moves to a public area, a public event begins, an item appears on a shelf) are publishable as Protocol events. The `PublishProtocolEvent` tool and `Protocol\BeatPublisher` listener handle this.

Three visibility levels, set per `PlaceBinding`:

| Visibility | Behavior |
|---|---|
| `private` | Beats stay inside the Server. Used for training, private games, personal journals. |
| `server` | Beats broadcast to the Server's Hub room — other visitors at the Place see them. |
| `federated` | Beats publish as Protocol events; neighboring Servers may subscribe. Used for public events, shared lore worlds. |

This is what makes the Story Engine a *citizen* of the spatial web rather than a chatbot bolted onto it.

---

## 13. Streaming, Queues, Performance

| Concern | Approach |
|---|---|
| Real-time narration to a Browser | SSE or Reverb broadcasting `BeatGenerated`. |
| Multiplayer turn arbitration | Beats published to a Colyseus Hub room via `Protocol\HubBridge`. |
| Background scene planning | Choreographer/Chronicler dispatched via AI SDK `queue()`. |
| Long campaigns | Chronicler compresses every N beats; Director sees `summary + window`. |
| Token budget | `Support\ContextBudget` trims oldest beats first, then lore hits. |
| Provider failover | AI SDK's array-of-providers; default `[anthropic, openai]`. |
| Determinism in tests | `Story::fake()` + seeded RNG + `Agent::fake()` with scripted tool sequences. |

---

## 14. Security & Safety

1. **Tool scoping** — all tools accept Campaign in constructor; never trust IDs from prompts. (Laravel AI SDK best practice.)
2. **Mutation allowlist** — Ruleset declares which Avatar fields are mutable; PHP enforces, not the prompt.
3. **No raw DB tool** — every state change goes through an Action.
4. **Secrets stay server-side** — `AvatarRole::secrets` is filtered out of payloads sent to the LLM unless explicitly opted in for that field.
5. **Prompt-injection hardening** — Player input wrapped in `<player_input>` tags; the Director's instructions explicitly state that content within those tags is dialogue, not commands.
6. **Mandatory safety preamble** — `Support\SafetyPreamble` is prepended to every Director prompt. The default preamble forbids: harm to real persons, sexualization of minors, weaponized PII, content the Place owner has not opted into. Operators *extend* the preamble; they cannot remove the floor.
7. **PII in training contexts** — Training-mode Campaigns log a warning at start when configured with non-Anthropic providers; the Server operator owns the decision.
8. **Federated publication consent** — a beat is only published to neighbors if both the PlaceBinding visibility and the Chronicler's `protocol_publishable` flag agree.

---

## 15. Events & Observability

All events fire through standard Laravel event infrastructure, plus AI SDK agent events (`AgentPrompted`, `ToolCalled`, etc.).

`php artisan story-engine:trace {campaign}` prints a chronological transcript of beats, tool calls, dice rolls, and Protocol publications — the audit trail Server operators need.

---

## 16. Configuration

`config/story-engine.php` (excerpt):

```php
return [
    'default_provider' => env('STORY_ENGINE_PROVIDER', 'anthropic'),

    'models' => [
        'director'      => env('STORY_DIRECTOR_MODEL', 'claude-sonnet-4-6'),
        'choreographer' => env('STORY_CHOREO_MODEL', 'claude-haiku-4-5-20251001'),
        'chronicler'    => env('STORY_CHRON_MODEL', 'claude-haiku-4-5-20251001'),
    ],

    'providers' => ['anthropic', 'openai'],

    'memory' => [
        'recent_beats_window'      => 30,
        'summarize_every_n_beats'  => 25,
        'max_lore_hits'            => 4,
    ],

    'safety' => [
        'preamble' => 'default',          // or path to a custom preamble file
        'redact_secrets' => true,
    ],

    'streaming' => [
        'driver' => env('STORY_STREAM', 'sse'),  // sse | reverb | hub | none
    ],

    'protocol' => [
        'default_visibility' => 'private',  // private | server | federated
        'hub_driver' => null,               // bind a HubBridge implementation
    ],

    'rulesets' => [
        'generic' => \StreetMesh\StoryEngine\Rulesets\GenericNarrativeRuleset::class,
        'd20'     => \StreetMesh\StoryEngine\Rulesets\D20Ruleset::class,
        'rubric'  => \StreetMesh\StoryEngine\Rulesets\RubricRuleset::class,
    ],

    'modes' => [
        'game'      => \StreetMesh\StoryEngine\Modes\GameMode::class,
        'training'  => \StreetMesh\StoryEngine\Modes\TrainingMode::class,
        'concierge' => \StreetMesh\StoryEngine\Modes\ConciergeMode::class,
    ],

    'place_binder' => null,  // host Server registers its own implementation

    'routes' => [
        'enabled'    => true,
        'prefix'     => 'api/story',
        'middleware' => ['api', 'auth:sanctum'],
    ],

    'packs' => [
        // Register Content Packs here
    ],
];
```

---

## 17. Testing Strategy

1. Unit tests per Tool, Action, Ruleset, Mode.
2. Feature tests using `Agent::fake()` with scripted tool-call sequences.
3. Snapshot tests on `PromptComposer` and `SafetyPreamble`.
4. Determinism mode for full end-to-end campaign replay in CI.
5. **Protocol contract tests** — verify that beats published with `federated` visibility match the StreetMesh Protocol's expected schema.

---

## 18. Build Order

### Milestone 1 — Skateboard (single-Place improv, no rules, no Protocol)
1. Package skeleton, ServiceProvider, `story-engine:install` and `:publish` commands.
2. Migrations (worlds, place_bindings, campaigns, scenes, beats, parties).
3. `GenericNarrativeRuleset` + `GameMode`.
4. `DirectorAgent` with `ProposeBeat`, `RequestPlayerChoice`, `RecallLore`, `DescribePlace`.
5. `Story` facade with bind / start / advance.
6. Stub `PlaceBinder` (returns a hardcoded extent — real Server provides its own).
7. Feature test: 5-beat improv at a Place, end-to-end with `Agent::fake()`.

### Milestone 2 — Bicycle (multi-scene, memory, Avatar integration)
8. `ChoreographerAgent` + `ChroniclerAgent` + queued summarization.
9. `LoreEntry` + SimilaritySearch RAG.
10. `MeshObjectReader` — pull lore from schema.org microdata.
11. Integration with `streetmesh/avatars`: `AvatarRole` model, `GetAvatar` tool.
12. `EndScene` + scene transitions.
13. SSE streaming.

### Milestone 3 — Motorcycle (rules, dice, modes)
14. `D20Ruleset` + `RubricRuleset`.
15. `RollDice`, `ResolveCheck`, `UpdateAvatarState`.
16. `TrainingMode`, `ConciergeMode` with mode-restricted toolsets.
17. Determinism + replay harness.

### Milestone 4 — Car (Protocol + content packs + frontend)
18. `BeatPublisher` + `HubBridge` for federated/multiplayer publication.
19. `ContentPack` contract + `make:story-pack` Artisan command.
20. Reference packs: `fantasy-osr`, `training-scenario`, `retail-concierge`.
21. Frontend packages: `@streetmesh/story-vue`, `@streetmesh/story-react`, `@streetmesh/story-alpine` — beat stream renderer + choice prompter.
22. `DESIGN.md` + cookbook.

---

## 19. Open Questions to Resolve in the First Claude Code Session

1. **PHP / Laravel versions.** Avatars uses PHP 8.2+, Laravel 11.x or 12.x. Match this. (AI SDK requires Laravel 12+.)
2. **Vector store.** pgvector preferred (matches Laravel AI SDK's SimilaritySearch); document SQLite/MySQL fallback.
3. **Avatar coupling strength.** Hard dependency on `streetmesh/avatars`, or soft via interface? Recommendation: **soft** — a `ResolvesAvatars` contract that `streetmesh/avatars` implements, with a stub for standalone use. Mirrors how the Server reference impl can swap pieces.
4. **Place ID format.** URN (`streetmesh://server.example/place/apothecary`) or opaque string? Recommendation: opaque string at the engine level, URN at the Protocol boundary.
5. **Hub driver.** Colyseus is referenced by the org but is a separate package (`StreetMesh/Hub`). Make `HubBridge` an interface; ship a Colyseus implementation in a *separate* sub-package or example.
6. **License clarity.** The framework org-level docs are CC BY-NC-SA 4.0; sibling code packages (Avatars) are MIT. Story Engine = **MIT**, with a `NOTICE` file pointing to the framework's CC license for any prose/lore drawn from `StreetMesh/Experience`.

---

## 20. North Star

> *A Place without a story is an empty room.*

The Story Engine is the smallest installable thing that turns a StreetMesh Place into a *somewhere*. It speaks Avatar, it speaks Place, it speaks Protocol. It hosts games, runs trainings, welcomes shoppers, and narrates events — all from one engine, one Composer install, one set of conventions consistent with every other StreetMesh package.

It belongs to no one and everyone. It is humane by default, modular by design, and decentralized by birth.

It is one more brick in The Dream.
