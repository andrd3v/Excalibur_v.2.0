# Excalibur_v.2.0
## Second version of Excalibur!

<p align="center">
  <img src="https://github.com/andrd3v/Excalibur_v.2.0/blob/main/1.jpg" width="45%">
  <img src="https://github.com/andrd3v/Excalibur_v.2.0/blob/main/2.jpg" width="45%">
</p>


This is a simple example demonstrating the functionality of Excalibur v2. It served as the basis for creating various cheats for Standoff 2, Critical Ops, and other games.<br>

The main ESP rendering logic is implemented in the update_data method. On git simple demo, not real code for some kind of game. [HERE](https://github.com/andrd3v/Excalibur_v.2.0/blob/main/esp/drawing_view/esp.mm#L101)<br>

## How to use?
**1. Get the PID and task port of your game:**
```
pid_t pid = get_pid_by_name("MainBinaryOfGame");
task_t task = get_task_by_pid(pid);
// OR
// task_t task = get_task_for_PID(pid); <- ezz detect
```

**2. Get the UnityFramework base address:**
```
mach_vm_address_t unity_base_addr = get_image_base_address(task, "UnityFramework");
```

**3. Get your players and their transforms.**
U can use static typeinfo for this [SEE HERE](https://iosgods.com/topic/70716-static-members-and-multithreading/), example:<br>
```
mach_vm_address_t typeinfo_addr = Read<mach_vm_address_t>(unity_base_addr + HERE_ADDR_OF_TYPE_INFO, task);
mach_vm_address_t statik_addr = Read<mach_vm_address_t>(typeinfo_addr + 0xb8, task); // may need to change offset
void* statik_pole = Read<void*>(statik_addr, task);
```

**4. After jumping through fields and finding the player list, you can read it like this:**
```c
mach_vm_address_t list_ptr = Read<mach_vm_address_t>((mach_vm_address_t)game_manager + 0x10, task);
int list_length = Read<int>(list_ptr + 0x18, task);
mach_vm_address_t internal_ptr = Read<mach_vm_address_t>(list_ptr + 0x10, task);

mach_vm_address_t players[10];
mach_vm_size_t out_size = 0;

mach_vm_read_overwrite(
    task,
    internal_ptr + 0x20,
    list_length * sizeof(mach_vm_address_t),
    (mach_vm_address_t)players,
    &out_size
);

std::vector<mach_vm_address_t> charVector(players, players + 10);
```

**5. Once you have a player, get their transform:**
```c
NSMutableArray<NSValue *> *boxesMutable = [NSMutableArray array];
NSMutableArray<NSDictionary *> *playersMutable = [NSMutableArray array];

for (size_t i = 0; i < charVector.size(); i++)
{
    mach_vm_address_t pl = charVector[i];
    // .....
    mach_vm_address_t trnsfrm_spine = Read<mach_vm_address_t>(pl + 0x120, task);
}
```

**6. Then you can get the player's position:**
```c
Vector3 vec3_pos_trnsfrm_spine = get_position_by_transform(trnsfrm_spine, task);
if (vec3_pos_trnsfrm_spine.x == 0.0f && vec3_pos_trnsfrm_spine.y == 0.0f && vec3_pos_trnsfrm_spine.z == 0.0f)
    continue;
// change addrs in get_position_by_transform
```

**7. You can get the world-to-screen coordinates like this:**
```c
Vector3 top_w2s = WorldToScreen(vec3_pos_trnsfrm_spine, camera, self.bounds.size.width, self.bounds.size.height, task); // get Camera at ur self
Vector3 bottom_w2s = WorldToScreen(feet_wvec3_pos_trnsfrm_spineorld, camera, self.bounds.size.width, self.bounds.size.height, task); // get Camera at ur self
// feet_wvec3_pos_trnsfrm_spineorld = vec3_pos_trnsfrm_spine - 1.8f * 0.5f;
// change addrs in WorldToScreen
```

**8. Creating a box:**
```objc
ESPBox box;
box.pos = bottom_w2s;
box.pos.x = bottom_w2s.x - (width / 2.0f);
box.pos.y = bottom_w2s.y - height;
box.pos.z = 0.0f;
box.width = width;
box.height = height;

NSValue *val = [NSValue valueWithBytes:&box objCType:@encode(ESPBox)];
[boxesMutable addObject:val];
```

**9. Render the boxes outside the loop:**
```objc
dispatch_async(dispatch_get_main_queue(), ^{
    [self setBoxes:boxesMutable];
});
```

### Archive Note
This code was originally written in May 2025 and demonstrates the ESP implementation for Standoff 2 / Critical Ops / Coll of Duty / etc....<br>
