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
