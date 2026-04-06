--// Cam + Boneco Lock (By light_use182)
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local camEnabled = false
local lockedTarget = nil 

--// 1. FUNÇÃO DE NOTIFICAÇÃO (Exibe apenas a sua assinatura)
local function showCredit()
    local screenGui = LocalPlayer:WaitForChild("PlayerGui"):FindFirstChild("AimCredit") or Instance.new("ScreenGui", LocalPlayer.PlayerGui)
    screenGui.Name = "AimCredit"
    
    local msg = Instance.new("TextLabel")
    msg.Size = UDim2.new(0, 400, 0, 50)
    msg.Position = UDim2.new(0.5, -200, 0.4, 0) -- Centro da tela
    msg.BackgroundTransparency = 1
    msg.Text = "By light_use182"
    msg.TextColor3 = Color3.fromRGB(255, 255, 255)
    msg.Font = Enum.Font.GothamBold
    msg.TextSize = 25
    msg.Parent = screenGui
    
    -- Some após 2 segundos
    task.delay(2, function()
        msg:Destroy()
    end)
end

-- Chama a frase logo na execução do script
showCredit()

--// 2. FUNÇÃO DE ARRASTAR
local function makeDraggable(gui)
    local dragging, dragInput, dragStart, startPos
    gui.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true; dragStart = input.Position; startPos = gui.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then dragging = false end
            end)
        end
    end)
    gui.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then dragInput = input end
    end)
    UIS.InputChanged:Connect(function(input)
        if dragging and input == dragInput then
            local delta = input.Position - dragStart
            gui.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
end

--// 3. FUNÇÃO DE BUSCA PELO CENTRO DA TELA
local function getTargetInFocus()
    local closestToCenter = nil
    local shortestDistance = math.huge
    local centerScreen = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            local hum = p.Character:FindFirstChildOfClass("Humanoid")
            if hum and hum.Health > 0 then
                local screenPos, onScreen = Camera:WorldToViewportPoint(p.Character.HumanoidRootPart.Position)
                if onScreen then
                    local distToCenter = (Vector2.new(screenPos.X, screenPos.Y) - centerScreen).Magnitude
                    if distToCenter < shortestDistance then
                        shortestDistance = distToCenter
                        closestToCenter = p.Character.HumanoidRootPart
                    end
                end
            end
        end
    end
    return closestToCenter
end

--// 4. UI DO BOTÃO
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "AimLockFinal"
ScreenGui.Parent = CoreGui

local aimBtn = Instance.new("TextButton")
aimBtn.Size = UDim2.new(0, 50, 0, 50)
aimBtn.Position = UDim2.new(0.5, -25, 0.1, 0)
aimBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
aimBtn.Text = "aim"
aimBtn.TextColor3 = Color3.new(1, 1, 1)
aimBtn.Font = Enum.Font.GothamBold
aimBtn.Parent = ScreenGui
Instance.new("UICorner", aimBtn).CornerRadius = UDim.new(1, 0)
makeDraggable(aimBtn)

--// 5. ATIVAÇÃO
aimBtn.MouseButton1Click:Connect(function()
    camEnabled = not camEnabled
    
    if camEnabled then
        lockedTarget = getTargetInFocus()
        if lockedTarget then
            aimBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
            Camera.CameraType = Enum.CameraType.Scriptable
        else
            camEnabled = false
        end
    else
        lockedTarget = nil
        aimBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
        Camera.CameraType = Enum.CameraType.Custom
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
            LocalPlayer.Character:FindFirstChildOfClass("Humanoid").AutoRotate = true
        end
    end
end)

--// 6. LOOP DE MANUTENÇÃO
RunService.RenderStepped:Connect(function()
    if camEnabled and lockedTarget and LocalPlayer.Character then
        local myHRP = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        local humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        
        if not lockedTarget.Parent or (lockedTarget.Parent:FindFirstChildOfClass("Humanoid") and lockedTarget.Parent:FindFirstChildOfClass("Humanoid").Health <= 0) then
            camEnabled = false
            lockedTarget = nil
            aimBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
            Camera.CameraType = Enum.CameraType.Custom
            return
        end

        if myHRP and humanoid then
            humanoid.AutoRotate = false
            local targetPos = lockedTarget.Position
            local lookAtBody = Vector3.new(targetPos.X, myHRP.Position.Y, targetPos.Z)
            myHRP.CFrame = CFrame.lookAt(myHRP.Position, lookAtBody)
            local camPos = myHRP.Position + (myHRP.CFrame.LookVector * -10) + Vector3.new(0, 5, 0)
            Camera.CFrame = CFrame.lookAt(camPos, targetPos)
        end
    end
end)
