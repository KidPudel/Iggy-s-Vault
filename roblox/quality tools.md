[My advice](https://devforum.roblox.com/t/most-preferred-best-frameworks/2823121/3) would be not to rely on a “Framework” but use a variety of quality tools that get the job done.

---

In my main projects I use the following open source assets:

- Saving Player Data: [Profile Service](https://devforum.roblox.com/t/save-your-player-data-with-profileservice-datastore-module/667805)
- Replicating Player Data: [Replica Service](https://devforum.roblox.com/t/replicate-your-states-with-replicaservice-networking-system/894736)

_ProfileService and ReplicaService compliment each other very well._

**Other Datastore Modules**

- [Suphi’s Datastore Module](https://devforum.roblox.com/t/suphis-datastore-module/2425597)
- [ProfileStore (Successor of ProfileService)](https://devforum.roblox.com/t/profilestore-save-your-player-data-easy-datastore-module/3190543)

_– You don’t have to use a datastore module however if your game has trading or important player data then I suggest to use them._

**Admin Control**

- [Cmdr](https://devforum.roblox.com/t/cmdr-a-fully-extensible-and-type-safe-command-console-for-roblox-developers/182815). It has great intellisense and customizability.

– You can use admin guis too, there are plenty. I use Cmdr because it’s more than an Admin panel and helps a lot with testing your game for bugs!

**Replication**

- Complex replication; [ByteNet](https://devforum.roblox.com/t/bytenet-advanced-networking-library-w-buffer-serialization-strict-luau-absurd-optimization-and-rbxts-support-043/2733365), for sending large quantities of data in a small period of time.
- General networking: Use standard Remote Events/Functions, I also Made a custom Remote module, which I may opensource later.

_– If your game is simple then standard remotes are perfect. FPS Shooters or high performance games may need a networking module (buffer module)_

**Visual**

- For games that have different zones, I use [ZonePlus](https://devforum.roblox.com/t/zoneplus-v320-construct-dynamic-zones-and-effectively-determine-players-and-parts-within-their-boundaries/1017701). Really easy to use. (Don’t have to deal with touched events/ people leaving)
- CameraShaker: [GitHub - Sleitnick/RbxCameraShaker: Camera shake effects for Roblox games](https://github.com/Sleitnick/RbxCameraShaker) (Used in DOORS)
- UI Emitter Module: [UIEmitter Module - UI Particles (& Confetti)](https://devforum.roblox.com/t/uiemitter-module-ui-particles-confetti/2913477)  This module is really cool for UI Particles.
- ProximityPrompts: [Proximity Prompt Customizer by Bitwise](https://devforum.roblox.com/t/proximity-prompt-customizer/1663458), really easy to use and customize.

**Audio**

- Most my games dont have that much SFX so I use a simple audio module.  
    – Here’s the link: [https://create.roblox.com/store/asset/96307808599298/AudioPlayer-Modded](https://create.roblox.com/store/asset/96307808599298/AudioPlayer-Modded)

_– This module was modded from a previous Roblox template, it has preloading and effect playing functions too. (I mainly use it for BGM)_

**(Other) Utilities**

- Signal: [MadworkScriptSignal](https://github.com/MadStudioRoblox/ReplicaService/blob/master/src/ReplicatedStorage/MadworkScriptSignal.lua)
- Maid: [MadworkMaid](https://github.com/MadStudioRoblox/ReplicaService/blob/master/src/ReplicatedStorage/MadworkMaid.lua)

---

There are many free resources in [#resources:community-resources](https://devforum.roblox.com/c/resources/community-resources/74) . A lot of them were compiled into a list: [# | best posts - free/paid | #](https://devforum.roblox.com/t/best-posts-freepaid/3117786). Some of these are paid but they are still awesome 
## **Roblox Toolbox (In-Studio)**

Search these terms in the Creator Marketplace:

- "Tycoon Kit"
- "Simulator Framework"
- "Pet System"
- "Rebirth System"
- "Dropper System"

Quality varies wildly. Inspect scripts before using — some are outdated or have malicious code.