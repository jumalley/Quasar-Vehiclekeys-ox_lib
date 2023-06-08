# Quasar-Vehiclekeys-ox_lib

**Qs-Vehiclekeys** using **ox_lib** skill check + progressCircle

**hotwire.lua**

```lua
function MiniGameHotWire(state)
    --Example https://github.com/Utinax/reload-skillbar
    local finished = lib.skillCheck({'easy', 'easy', {areaSize = 60, speedMultiplier = 2}, 'hard'}, {'e'})
    if finished then
        inHotwire = true
        inMinigame = false
        initKeys(state)
    else
        inHotwire = false
        inMinigame = false
        Hotwire(state)
    end
end
```

```lua
function CustomProgBarsHotWire(plates , vehicle_model, coords, time)
if lib.progressCircle({
    duration = time,
    position = 'bottom',
    label = 'Récupéraration des clés',
    useWhileDead = false,
    canCancel = true,
    disable = {
        car = true,
    },
    anim = {
        dict = 'anim@amb@clubhouse@tutorial@bkr_tut_ig3@',
        clip = 'machinic_loop_mechandplayer'
    },
})
then
TriggerServerEvent(Config.Eventprefix..':server:givekey', plates, vehicle_model)
ClearPedTasks(PlayerPedId())
Citizen.Wait(5000)
TriggerServerEvent(Config.Eventprefix..':server:notifyCops', coords)
else
lib.notify({
    title = 'Vous avez annulé',
    description = 'Vous avez annulé votre action en cours.',
    type = 'error'
})
end
```

```lua
function CustomProgBarsDeadPed(plates , vehicle_model)
if lib.progressCircle({
    position = 'bottom',
    duration = 10000,
    label = Lang("VEHICLEKEYS_HOTWIRE_STEAL"),
    useWhileDead = false,
    canCancel = true,
    disable = {
        car = true,
    },
    anim = {
        dict = 'missheistdockssetup1clipboard@idle_a',
        clip = 'idle_a'
    },
})
then
ClearPedTasks(PlayerPedId())
TriggerServerEvent(Config.Eventprefix..':server:givekey', plates, vehicle_model)
else
lib.notify({
    title = 'Vous avez annulé',
    description = 'Vous avez annulé votre action en cours.',
    type = 'error'
})
end
end
```

**minigame.lua**

```lua
function LockpickDoorAnim(time)
    local ped = PlayerPedId()
    time = time / 1000
    loadAnimDict("veh@break_in@0h@p_m_one@")
    TaskPlayAnim(ped, "veh@break_in@0h@p_m_one@", "low_force_entry_ds" ,3.0, 3.0, -1, 16, 0, false, false, false)
    openingDoor = true
    CreateThread(function()
        while openingDoor do
            TaskPlayAnim(ped, "veh@break_in@0h@p_m_one@", "low_force_entry_ds", 3.0, 3.0, -1, 16, 0, 0, 0, 0)
            Wait(1000)
            time = time - 1
            if time <= 0 then
                openingDoor = false
                StopAnimTask(ped, "veh@break_in@0h@p_m_one@", "low_force_entry_ds", 1.0)
            end
        end
    end)
end
```

```lua
function lockpickFinish(success)
    local ped = PlayerPedId()
    local coords = GetEntityCoords(ped)
    local vehicle = GetClosestVehicle()
    local driver = GetPedInVehicleSeat(vehicle, -1)
    if success then 
        if math.random(1, 100) <= Config.LockpickFail then
            finishLockpick = true
            print('finishLockpick', finishLockpick)
            TriggerServerEvent(Config.Eventprefix..':server:setVehLockState', NetworkGetNetworkIdFromEntity(vehicle), 1)
            StopAnimTask(ped, "veh@break_in@0h@p_m_one@", "low_force_entry_ds", 1.0)
            SendTextMessage(Lang("VEHICLEKEYS_NOTIFICATION_LOCKPICK_SUCCESS"), 'success')
            PlayVehicleDoorOpenSound(vehicle, 0)
            TriggerServerEvent(Config.Eventprefix..':server:notifyCops', coords)  
    
            if Config.LockpickAlarm then if math.random(1, 100) < Config.StartAlarmChance then startAlarm(vehicle) end end
            if math.random(1, 100) < Config.LockpickBrokenChance then
                SendTextMessage(Lang("VEHICLEKEYS_NOTIFICATION_LOCKPICK_BROKEN"), 'error')
                TriggerServerEvent(Config.Eventprefix..":server:RemoveLockpick", Config.LockpickItem, 1)
                if Config.Debug then
                    TriggerServerEvent(Config.Eventprefix..':server:serverLog', '[qs-vehiclekeys] A lockpick attempt failed on: '..coords)
                end
            end
            finishLockpick = true
            openingDoor = false
            IsHotwiring = false
            Wait(20000)
            finishLockpick = false
        else
            openingDoor = false
            IsHotwiring = false
            StopAnimTask(ped, "veh@break_in@0h@p_m_one@", "low_force_entry_ds", 1.0)
            SendTextMessage(Lang("VEHICLEKEYS_NOTIFICATION_LOCKPICK_FAIL"), 'error')
            PlayVehicleDoorOpenSound(vehicle, 0)
            startAlarm(vehicle)
            TriggerServerEvent(Config.Eventprefix..':server:notifyCops', coords) 
        end
    else
        openingDoor = false
        IsHotwiring = false
        StopAnimTask(ped, "veh@break_in@0h@p_m_one@", "low_force_entry_ds", 1.0)
        SendTextMessage(Lang("VEHICLEKEYS_NOTIFICATION_LOCKPICK_FAIL"), 'error')
        PlayVehicleDoorOpenSound(vehicle, 0)
        startAlarm(vehicle)
        TriggerServerEvent(Config.Eventprefix..':server:notifyCops', coords) 
    end
end
```
