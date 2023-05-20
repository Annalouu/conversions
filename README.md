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

Core = Config.Framework ~= 'none' and GetCoreObject() or nil


- Getting PlayerData -- Gets the players data

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


- Getting the PlayerUnload Event -- When the player unloads.

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


- Getting the Job Event -- When the job gets updated.

---@return string JobEvent The event to listen for when the player changes job
function GetJobEvent()
  if IsDuplicityVersion() then return end
  if Config.Framework == 'esx' then
    return 'esx:setJob'
  elseif Config.Framework == 'qb' then
    return 'QBCore:Client:OnJobUpdate'
  end
end


- Triggering CallBacks -- Trigger callbacks in each framework

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

- Get Entity Coords -- Uses the framework get entity coords

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


- Create Vehicle -- creates vehicles

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


- Creating Server CallBacks -- Creates server Callbacks

function CreateCallback(...)
  if not IsDuplicityVersion() then return end
  if Config.Framework == 'esx' then
    return Core.RegisterServerCallback(...)
  elseif Config.Framework == 'qb' then
    return Core.Functions.CreateCallback(...)
  end
end


- Create additem functions

function Additem(item, count, boolean)
  local Core = GetCoreObject()
  local Player = GetPlayerData(source)
  if not IsDuplicityVersion() then return end
    if Config.traphouse.Framework == 'esx' then
       Player.addInventoryItem(item, count)
    elseif Config.traphouse.Framework == 'qb' then
      Player.Functions.AddItem(item, count,boolean) 
      TriggerClientEvent('inventory:client:ItemBox', source, Core.Shared.Items[item], "add")
    end
end


---@param source number|nil The source of the player
---@return table PlayerData The player data of the player
function GetPlayerData(source)
  local Core = GetCoreObject()
  if IsDuplicityVersion() then
    if Config.traphouse.Framework == 'esx' then
      return Core.GetPlayerFromId(source)
    elseif Config.traphouse.Framework == 'qb' then
      return Core.Functions.GetPlayer(source)
    end
  else
    if Config.Framework == 'esx' then
      return Core.GetPlayerData()
    elseif Config.Framework == 'qb' then
      return Core.Functions.GetPlayerData()
    end
  end
end


function Removeitem(item, count, boolean)
  local Core = GetCoreObject()
  local Player = GetPlayerData(source)
  if not IsDuplicityVersion() then return end
    if Config.Framework == 'esx' then
      Player.addInventoryItem(item, count)
    elseif Config.Framework == 'qb' then
      Player.Functions.RemoveItem(item, count,boolean) 
      TriggerClientEvent('inventory:client:ItemBox', source, Core.Shared.Items[item], "remove")
    end
end

function RemoveMoney(moneytype, count)
  local Core = GetCoreObject()
  local Player = GetPlayerData(source)
  if not IsDuplicityVersion() then return end
    if Config.Framework == 'esx' then
      return Player.RemoveMoney(count)
    elseif Config.Framework == 'qb' then
      return Player.Functions.RemoveMoney(moneytype, count)
    end
end

function Addmoney(moneytype, count)
  local Core = GetCoreObject()
  local Player = GetPlayerData(source)
  if not IsDuplicityVersion() then return end
    if Config.Framework == 'esx' then
      return Player.addMoney(count)
    elseif Config.Framework == 'qb' then
      return Player.Functions.AddMoney(moneytype, count)
    end
end


--Client side Notifications

function Notify(text,option)
  local Core = GetCoreObject()
  if IsDuplicityVersion() then return end
  if Config.traphouse.Notify == 'qb' then
    return Core.Functions.Notify(text, option)
  elseif Config.traphouse.Notify == 'ox' then
    return lib.notify({
      title = Config.traphouse.Title,
      description = text,
      type = option
  })
  elseif Config.traphouse.Notify == 'esx' then
    TriggerServerEvent('your-script:server:esxnotification', text, option)
  end
end

--For esx you must send a trigger to your Server side.
RegisterNetEvent('your-script:server:esxnotification', function(text,option)
  local src = source
  TriggerClientEvent('esx:showNotification', src, text, option)
end)


