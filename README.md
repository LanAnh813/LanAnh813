- 👋 Hi, I’m @LanAnh813
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
LanAnh813/LanAnh813 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
/*
 * This file is part of the CitizenFX project - http://citizen.re/
 *
 * See LICENSE and MENTIONS in the root of the source tree for information
 * regarding licensing.
 */

#include "StdInc.h"
#include <jitasm.h>
#include "Hooking.h"

#include <CrossBuildRuntime.h>

static HookFunction hookFunction([] ()
{
	// 1737: no longer needed
	if (xbr::IsGameBuildOrGreater<1737>())
	{
		return;
	}

	// CPlayerGameStateDataNode write/log functions tend to 'crash' when a player ped 'lost' its phone metadata (offset CPed + 4256 in 331)
	// as tends to happen when changing to certain animal models. This is commonly 'worked around' by people 'willing to mod' by removing the
	// call to CNetObjPlayer::`IPlayerNodeDataAccessor`'s 'update game state' function, which will *obviously* break various other state elements
	// in said node, instead of just 'Mobile Ring State' (the culprit).

	// To fix this, we replace the code that handles this object with our own routine which also verifies if the value is set in the first place.
	static struct : public jitasm::Frontend
	{
		virtual void InternalMain() override
		{
			// is rcx null? if so, skip this entire thing
			test(rcx, rcx);
			je("skipRuntime");

			// replication of original game code
			movzx(eax, byte_ptr[rcx + 0x2D]);
			add(eax, -2);

			// compare
			cmp(eax, r13d);

			// skip to ending?
			jbe("skipRuntime");

			// update data
			movzx(eax, byte_ptr[rcx + 0x2C]);
			mov(dword_ptr[rdx + 0xF8], eax);

			// return label
			L("skipRuntime");
			ret();
		}
	} phoneHookRoutine;

	// find the pattern, nop the original code and add a call
	void* phoneHookPoint = hook::pattern("0F B6 41 2D 83 C0 FE 41 3B C5 76 0A").count(1).get(0).get<void>();
	hook::nop(phoneHookPoint, 22);
	hook::call(phoneHookPoint, phoneHookRoutine.GetCode());
});
/*
 * This file is part of the CitizenFX project - http://citizen.re/
 *
 * See LICENSE and MENTIONS in the root of the source tree for information
 * regarding licensing.
 */

#include "StdInc.h"
#include <jitasm.h>
#include "Hooking.h"

#include <CrossBuildRuntime.h>

static HookFunction hookFunction([] ()
{
	// 1737: no longer needed
	if (xbr::IsGameBuildOrGreater<1737>())
	{
		return;
	}

	// CPlayerGameStateDataNode write/log functions tend to 'crash' when a player ped 'lost' its phone metadata (offset CPed + 4256 in 331)
	// as tends to happen when changing to certain animal models. This is commonly 'worked around' by people 'willing to mod' by removing the
	// call to CNetObjPlayer::`IPlayerNodeDataAccessor`'s 'update game state' function, which will *obviously* break various other state elements
	// in said node, instead of just 'Mobile Ring State' (the culprit).

	// To fix this, we replace the code that handles this object with our own routine which also verifies if the value is set in the first place.
	static struct : public jitasm::Frontend
	{
		virtual void InternalMain() override
		{
			// is rcx null? if so, skip this entire thing
			test(rcx, rcx);
			je("skipRuntime");

			// replication of original game code
			movzx(eax, byte_ptr[rcx + 0x2D]);
			add(eax, -2);

			// compare
			cmp(eax, r13d);

			// skip to ending?
			jbe("skipRuntime");

			// update data
			movzx(eax, byte_ptr[rcx + 0x2C]);
			mov(dword_ptr[rdx + 0xF8], eax);

			// return label
			L("skipRuntime");
			ret();
		}
	} phoneHookRoutine;

	// find the pattern, nop the original code and add a call
	void* phoneHookPoint = hook::pattern("0F B6 41 2D 83 C0 FE 41 3B C5 76 0A").count(1).get(0).get<void>();
	hook::nop(phoneHookPoint, 22);
	hook::call(phoneHookPoint, phoneHookRoutine.GetCode());
});
