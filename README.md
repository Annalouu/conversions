Made by Donhulio

---------SHARED
- Getting the Object

```lua
local Core = nil
---@return table Core The core object of the framework
function GetCoreObject()
  if Config.Framework == 'esx' then
    if not Core then
      Core = exports['es_extended']:getSharedObject()
    end
  elseif Config.Framework == 'qb' then
    if not Core then
      Core = exports['qb-core']:GetCoreObject()
    end
  end
  return Core
end

Core = Config.Framework ~= 'none' and GetCoreObject() or nil```

- Getting PlayerData

---@return string LoadEvent The event to listen for when the player loads
function GetLoadEvent()
  if IsDuplicityVersion() then return end
  if Config.Framework == 'esx' then
    return 'esx:playerLoaded'
  elseif Config.Framework == 'qb' then
    return 'QBCore:Client:OnPlayerLoaded'
  elseif Config.Framework == 'none' then
    return 'onClientResourceStart'
  end
end

- Getting the PlayerUnload Event


---@return string UnloadEvent The event to listen for when the player unloads
function GetUnloadEvent()
  if IsDuplicityVersion() then return end
  if Config.Framework == 'esx' then
    return 'esx:onPlayerLogout'
  elseif Config.Framework == 'qb' then
    return 'QBCore:Client:OnPlayerUnload'
  elseif Config.Framework == 'none' then
    return 'onClientResourceStop'
  end
end

- Getting the Job Event


---@return string JobEvent The event to listen for when the player changes job
function GetJobEvent()
  if IsDuplicityVersion() then return end
  if Config.Framework == 'esx' then
    return 'esx:setJob'
  elseif Config.Framework == 'qb' then
    return 'QBCore:Client:OnJobUpdate'
  end
end

- Triggering CallBacks


---@param name string The name of the event to trigger
---@param cb function The callback to call when the event is triggered
---@param ... any The arguments to pass to the event
---@return any The return value of the event
function TriggerCallback(name, cb, ...)
  if IsDuplicityVersion() then return end
  if Config.Framework == 'esx' then
    return Core.TriggerServerCallback(name, cb, ...)
  elseif Config.Framework == 'qb' then
    return Core.Functions.TriggerCallback(name, cb, ...)
  end
end


---SERVER

- Get Entity Coords


---@param entity number The entity to get the coords from
---@return vector4 coords The coords of the entity
function GetCoords(entity)
  if not IsDuplicityVersion() then return end
  if Config.Framework == 'esx' or Config.Framework == 'none' then
    local coords = GetEntityCoords(entity)
    local heading = GetEntityHeading(entity)
    return vector4(coords.x, coords.y, coords.z, heading)
  elseif Config.Framework == 'qb' then
    return Core.Functions.GetCoords(entity)
  end
end

- Create Vehicle 


function Sv_CreateVehicle(source, model, coords)
  if not IsDuplicityVersion() then return end
  if Config.Framework == 'esx' or Config.Framework == 'none' then
    model = type(model) == 'string' and joaat(model) or model
    coords = coords or GetEntityCoords(GetPlayerPed(source))
    local CreateAutomobile = `CREATE_AUTOMOBILE`
    local veh = Citizen.InvokeNative(CreateAutomobile, model, coords, coords.w, true, true)
    while not DoesEntityExist(veh) do Wait(0) end
    return veh
  elseif Config.Framework == 'qb' then
    return Core.Functions.CreateVehicle(source, model, coords, false)
  end
end

- Creating Server CallBacks


function CreateCallback(...)
  if not IsDuplicityVersion() then return end
  if Config.Framework == 'esx' then
    return Core.RegisterServerCallback(...)
  elseif Config.Framework == 'qb' then
    return Core.Functions.CreateCallback(...)
  end
end
