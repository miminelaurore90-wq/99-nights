# 99-nights

--[[
Sistema de Ciclo Día-Noche para simular el paso del tiempo.
Este script debe colocarse en ServerScriptService.
Controla la luz ambiental y la hora del día usando la propiedad TimeOfDay del Lighting service.
]]

-- Servicios clave de Roblox
local Lighting = game:GetService("Lighting")
local RunService = game:GetService("RunService")

-- CONFIGURACIÓN DEL CICLO DE TIEMPO
local DAY_LENGTH_SECONDS = 300  -- Duración de un día completo en segundos (e.g., 5 minutos)
local TIME_SCALE = 1 / DAY_LENGTH_SECONDS -- Factor de escala para el paso del tiempo
local INITIAL_TIME = 6          -- Hora inicial (6 AM)
local MAX_TIME = 24             -- Máxima hora (24, o 0)

-- Horas clave (de 0 a 24)
local DAWN_START = 5    -- Empieza a amanecer
local DAY_START = 6     -- Día completo
local DUSK_START = 17   -- Empieza a anochecer
local NIGHT_START = 18  -- Noche completa (la hora peligrosa)

-- Estado del juego
local currentHour = INITIAL_TIME -- Variable para mantener la hora actual
local currentDay = 1             -- Contador de días

-- Función para actualizar la hora del día en el juego
local function updateTimeOfDay()
    -- Converte la hora actual (0-24) a formato de Roblox (ej: "06:00:00")
    local hours = math.floor(currentHour)
    local minutes = math.floor((currentHour - hours) * 60)
    
    local timeString = string.format("%02d:%02d:00", hours % 24, minutes)
    Lighting.TimeOfDay = timeString
end

-- Función para manejar eventos al llegar la noche
local function handleNightEvents()
    print(string.format("¡La Noche %d ha caído! ¡Prepárense para el ataque!", currentDay))
    
    -- *** LÓGICA DE EVENTOS DE JUEGO AQUÍ ***
    -- 1. Activación de oleadas de enemigos (Spawning/Manejo de AI)
    -- 2. Mostrar advertencia en la pantalla del jugador (GUI)
    -- 3. Cambiar la música o el sonido ambiental
    -- **************************************
end

-- Bucle principal del juego (se ejecuta constantemente)
RunService.Heartbeat:Connect(function(deltaTime)
    -- Calcula cuánto tiempo real (en horas de juego) ha pasado
    local timePassed = deltaTime * TIME_SCALE * MAX_TIME
    
    -- Guarda la hora antes de la actualización para detectar el cambio de día
    local previousHour = currentHour

    -- 1. Actualiza la hora actual
    currentHour = currentHour + timePassed

    -- 2. Comprueba si ha comenzado un nuevo día
    if currentHour >= MAX_TIME then
        currentHour = currentHour - MAX_TIME -- Reinicia el contador de 24 horas
        currentDay = currentDay + 1          -- Incrementa el número de día
        print(string.format("--- ¡El Día %d ha Amanecido! ---", currentDay))
    end
    
    -- 3. Verifica si la noche acaba de empezar (cruzó de DUSK a NIGHT)
    if previousHour < NIGHT_START and currentHour >= NIGHT_START then
        handleNightEvents()
    end

    -- 4. Aplica la nueva hora al entorno
    updateTimeOfDay()
end)

-- Inicialización
updateTimeOfDay()
print(string.format("Ciclo Día/Noche iniciado. Duración del día: %d segundos.", DAY_LENGTH_SECONDS)) 
--[[
Sistema de Daño Nocturno.
Este script aplica daño a los jugadores si están fuera del área segura
durante las horas peligrosas (la noche).

Debe ejecutarse en el servidor.
]]

-- Servicios clave de Roblox
local Lighting = game:GetService("Lighting")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")

-- CONFIGURACIÓN DEL SISTEMA DE DAÑO
local SAFE_ZONE_PART_NAME = "CampfireZone" -- ¡IMPORTANTE! Nombre de la parte que define el área segura
local SAFE_ZONE_RADIUS = 50                 -- Radio de la zona segura (en studs)
local NIGHT_DAMAGE_PER_SECOND = 10          -- Cantidad de daño por segundo durante la noche
local DAMAGE_INTERVAL = 1                   -- Frecuencia de chequeo de daño (segundos)

-- Horas clave (coinciden con el script DayNightCycle.lua)
local NIGHT_START = 18.0 -- 6 PM
local DAWN_START = 5.0   -- 5 AM

-- Variable para rastrear la última vez que se aplicó daño
local lastDamageTime = 0

-- Función auxiliar para obtener la parte de la zona segura
local function getSafeZoneCenter()
    -- Busca la parte llamada "CampfireZone" en el Workspace
    local safeZonePart = game.Workspace:FindFirstChild(SAFE_ZONE_PART_NAME)
    
    if not safeZonePart or not safeZonePart:IsA("BasePart") then
        warn("Advertencia: No se encontró la parte de la zona segura con el nombre '" .. SAFE_ZONE_PART_NAME .. "'. ¡El sistema de daño no funcionará!")
        -- Devuelve una posición por defecto (0, 0, 0) si no se encuentra
        return Vector3.new(0, 0, 0)
    end
    
    return safeZonePart.Position
end

-- Función para verificar si un jugador está fuera del área segura
local function isPlayerOutsideSafeZone(player, safeZoneCenter)
    local character = player.Character
    if not character then return false end

    local rootPart = character:FindFirstChild("HumanoidRootPart")
    if not rootPart then return false end

    -- Calcula la distancia desde el jugador hasta el centro de la zona segura
    local distance = (rootPart.Position - safeZoneCenter).Magnitude

    return distance > SAFE_ZONE_RADIUS
end

-- Función para verificar si es actualmente de noche
local function isNightTime()
    local timeParts = Lighting.TimeOfDay:split(":")
    local currentHour = tonumber(timeParts[1]) + (tonumber(timeParts[2]) / 60)
    
    -- La noche está activa si la hora es >= NIGHT_START O < DAWN_START
    return currentHour >= NIGHT_START or currentHour < DAWN_START
end

-- Bucle principal de chequeo de daño
RunService.Heartbeat:Connect(function(deltaTime)
    local currentTime = tick()
    
    -- Solo aplica daño a intervalos definidos
    if currentTime - lastDamageTime >= DAMAGE_INTERVAL then
        lastDamageTime = currentTime

        -- 1. Verifica si es de noche
        if isNightTime() then
            local safeZoneCenter = getSafeZoneCenter()

            -- 2. Itera sobre todos los jugadores
            for _, player in ipairs(Players:GetPlayers()) do
                -- 3. Verifica si el jugador está fuera de la zona segura
                if isPlayerOutsideSafeZone(player, safeZoneCenter) then
                    local humanoid = player.Character and player.Character:FindFirstChildOfClass("Humanoid")
                    
                    if humanoid and humanoid.Health > 0 then
                        -- 4. Aplica el daño
                        humanoid:TakeDamage(NIGHT_DAMAGE_PER_SECOND * DAMAGE_INTERVAL)
                        print(string.format("%s recibió %.1f de daño por estar fuera del campamento.", player.Name, NIGHT_DAMAGE_PER_SECOND * DAMAGE_INTERVAL))
                    end
                end
            end
        end
    end
end)

-- NOTA IMPORTANTE: Asegúrate de colocar un objeto "Part" en el Workspace llamado "CampfireZone"
-- para que el sistema funcione correctamente. Su posición y el radio (50 studs) definirán el área segura.
print("Sistema de daño nocturno iniciado. ¡Asegúrate de tener una parte llamada 'CampfireZone' en el Workspace!")
