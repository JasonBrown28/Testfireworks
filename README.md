CLIENT 

++++++ 

local fireworks = {} -- Table to store spawned fireworks

-- Function to spawn a firework
function SpawnFirework(coords)
    local firework = CreateObject("p_sparkler01x", coords.x, coords.y, coords.z, true, false, false)
    table.insert(fireworks, firework)
    return firework
end

-- Function to light and explode a firework
function LightAndExplodeFirework(firework)
    SetEntityAsMissionEntity(firework, true, true)
    local particleDict = "scr_indep_fireworks"
    local particleName = "scr_indep_firework_burst_spawn"
    RequestNamedPtfxAsset(particleDict)
    while not HasNamedPtfxAssetLoaded(particleDict) do
        Wait(0)
    end
    UseParticleFxAssetNextCall(particleDict)
    StartParticleFxLoopedOnEntity(particleName, firework, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0, 1.0, false, false, false)
    Wait(2000) -- Adjust the duration if needed
    table.remove(fireworks, firework)
    DeleteEntity(firework)
end

-- Event to receive firework spawn request from the server
RegisterNetEvent("spawnFirework")
AddEventHandler("spawnFirework", function(coords)
    local firework = SpawnFirework(coords)
    Wait(1000) -- Delay before auto-lighting the firework
    LightAndExplodeFirework(firework)
end)

-- Command to light all spawned fireworks
RegisterCommand("lightfireworks", function()
    for _, firework in pairs(fireworks) do
        LightAndExplodeFirework(firework)
    end
    TriggerEvent("chatMessage", "[Fireworks]", {255, 0, 0}, "All fireworks have been lit!")
end, false)


SERVER 

+++++

-- Command to request spawning a firework on the client
RegisterCommand("spawnfirework", function(source, args, rawCommand)
    local player = source
    local playerCoords = GetEntityCoords(GetPlayerPed(player))
    TriggerClientEvent("spawnFirework", player, playerCoords)
    TriggerClientEvent("chatMessage", player, "[Fireworks]", {255, 0, 0}, "A firework has been spawned!")
end, false)
