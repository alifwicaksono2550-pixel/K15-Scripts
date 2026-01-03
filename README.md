local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "K15 HUB",
   LoadingTitle = "Welcome " .. game.Players.LocalPlayer.DisplayName,
   LoadingSubtitle = "By Nocyz",
   ConfigurationSaving = {
      Enabled = true,
      FolderName = "K15HubConfig",
      FileName = "MainConfig"
   },
   KeySystem = false
})

Rayfield.Theme.Default.AccentColor = Color3.fromRGB(255, 105, 180)

local MainTab = Window:CreateTab("Main Script", 4483362458)
MainTab:CreateLabel("Welcome " .. game.Players.LocalPlayer.Name)

--- [ ESP PLAYER - RED COLOR ] ---
local ESP_Active = false
local function StartESP(Player)
    local Box = Drawing.new("Square")
    Box.Visible = false
    Box.Color = Color3.fromRGB(255, 0, 0) -- Merah Cerah
    Box.Thickness = 1
    Box.Filled = false

    local Tracer = Drawing.new("Line")
    Tracer.Visible = false
    Tracer.Color = Color3.fromRGB(255, 255, 255)
    Tracer.Thickness = 1

    local function Update()
        local Connection
        Connection = game:GetService("RunService").RenderStepped:Connect(function()
            if ESP_Active and Player.Character and Player.Character:FindFirstChild("HumanoidRootPart") and Player.Character:FindFirstChild("Humanoid") and Player.Character.Humanoid.Health > 0 then
                local RootPart = Player.Character.HumanoidRootPart
                local Position, OnScreen = workspace.CurrentCamera:WorldToViewportPoint(RootPart.Position)

                if OnScreen then
                    local Size = (workspace.CurrentCamera:WorldToViewportPoint(RootPart.Position - Vector3.new(0, 3, 0)).Y - workspace.CurrentCamera:WorldToViewportPoint(RootPart.Position + Vector3.new(0, 2.6, 0)).Y)
                    Box.Size = Vector2.new(Size * 0.7, Size)
                    Box.Position = Vector2.new(Position.X - Box.Size.X / 2, Position.Y - Box.Size.Y / 2)
                    Box.Visible = true

                    Tracer.From = Vector2.new(workspace.CurrentCamera.ViewportSize.X / 2, workspace.CurrentCamera.ViewportSize.Y)
                    Tracer.To = Vector2.new(Position.X, Position.Y + (Box.Size.Y / 2))
                    Tracer.Visible = true
                else
                    Box.Visible = false
                    Tracer.Visible = false
                end
            else
                Box.Visible = false
                Tracer.Visible = false
                if not Player.Parent or not ESP_Active then
                    Box:Remove()
                    Tracer:Remove()
                    Connection:Disconnect()
                end
            end
        end)
    end
    coroutine.wrap(Update)()
end

MainTab:CreateToggle({
   Name = "ESP Player",
   CurrentValue = false,
   Flag = "ESP",
   Callback = function(Value)
      ESP_Active = Value
      if Value then
          for _, v in pairs(game.Players:GetPlayers()) do
              if v ~= game.Players.LocalPlayer then StartESP(v) end
          end
          game.Players.PlayerAdded:Connect(function(plr)
              if ESP_Active then StartESP(plr) end
          end)
      end
   end,
})

--- [ CROSSHAIR - CENTERED ] ---
MainTab:CreateToggle({
   Name = "Crosshair (Dark Red X)",
   CurrentValue = false,
   Callback = function(Value)
      local pg = game.Players.LocalPlayer:WaitForChild("PlayerGui")
      if Value then
          local sg = Instance.new("ScreenGui", pg)
          sg.Name = "X_Crosshair"
          sg.IgnoreGuiInset = true 
          for i = 1, 2 do
              local f = Instance.new("Frame", sg)
              f.Size = UDim2.new(0, 2, 0, 14)
              f.BackgroundColor3 = Color3.fromRGB(139, 0, 0)
              f.Position = UDim2.new(0.5, 0, 0.5, 0)
              f.AnchorPoint = Vector2.new(0.5, 0.5)
              f.Rotation = (i == 1 and 45 or 135)
              f.BorderSizePixel = 0
          end
      else
          if pg:FindFirstChild("X_Crosshair") then pg.X_Crosshair:Destroy() end
      end
   end,
})

--- [ FLY SCRIPT ] ---
local Flying = false
local FlySpeed = 50
local BodyVel, BodyGyro
MainTab:CreateToggle({
   Name = "Fly",
   CurrentValue = false,
   Callback = function(Value)
      Flying = Value
      local char = game.Players.LocalPlayer.Character
      if not char or not char:FindFirstChild("HumanoidRootPart") then return end
      local root = char.HumanoidRootPart
      if Flying then
         BodyVel = Instance.new("BodyVelocity", root)
         BodyGyro = Instance.new("BodyGyro", root)
         BodyVel.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
         BodyVel.Velocity = Vector3.new(0, 0.1, 0)
         BodyGyro.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
         BodyGyro.P = 9000
         task.spawn(function()
            while Flying do
               if char.Humanoid.MoveDirection.Magnitude > 0 then
                   BodyVel.Velocity = workspace.CurrentCamera.CFrame.LookVector * FlySpeed
               else
                   BodyVel.Velocity = Vector3.new(0, 0.1, 0)
               end
               BodyGyro.CFrame = workspace.CurrentCamera.CFrame
               task.wait()
            end
            if BodyVel then BodyVel:Destroy() end
            if BodyGyro then BodyGyro:Destroy() end
         end)
      else
         if BodyVel then BodyVel:Destroy() end
         if BodyGyro then BodyGyro:Destroy() end
      end
   end,
})

MainTab:CreateSlider({
   Name = "Fly Speed",
   Range = {1, 500},
   Increment = 1,
   CurrentValue = 50,
   Callback = function(v) FlySpeed = v end,
})

--- [ WALKSPEED & JUMP ] ---
MainTab:CreateSlider({
   Name = "Walkspeed",
   Range = {16, 500},
   Increment = 1,
   CurrentValue = 16,
   Callback = function(v) if game.Players.LocalPlayer.Character:FindFirstChild("Humanoid") then game.Players.LocalPlayer.Character.Humanoid.WalkSpeed = v end end,
})

MainTab:CreateSlider({
   Name = "High Jump",
   Range = {50, 500},
   Increment = 1,
   CurrentValue = 50,
   Callback = function(v) 
      if game.Players.LocalPlayer.Character:FindFirstChild("Humanoid") then
         game.Players.LocalPlayer.Character.Humanoid.JumpPower = v 
         game.Players.LocalPlayer.Character.Humanoid.UseJumpPower = true
      end
   end,
})

--- [ ANTI FLING ] ---
MainTab:CreateToggle({
   Name = "Anti Fling",
   CurrentValue = false,
   Callback = function(v)
       _G.AF = v
       game:GetService("RunService").Stepped:Connect(function()
           if _G.AF then
               for _, p in pairs(game.Players:GetPlayers()) do
                   if p ~= game.Players.LocalPlayer and p.Character then
                       for _, part in pairs(p.Character:GetDescendants()) do
                           if part:IsA("BasePart") then part.CanCollide = false end
                       end
                   end
               end
           end
       end)
   end,
})

--- [ PERBAIKAN TELEPORT TO PLAYER ] ---
local TeleportDropdown = MainTab:CreateDropdown({
   Name = "Teleport to Player",
   Options = {"Select Player"}, -- Inisialisasi awal agar tidak kosong
   CurrentOption = {"Select Player"},
   MultipleOptions = false,
   Flag = "TPPlayer",
   Callback = function(Option)
      local targetName = Option[1]
      if targetName and targetName ~= "Select Player" then
          local target = game.Players:FindFirstChild(targetName)
          if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
              game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = target.Character.HumanoidRootPart.CFrame
          end
      end
   end,
})

-- Fungsi Update Daftar Nama Tanpa Menutup Dropdown
local function UpdatePlayerList()
    local pList = {"Select Player"}
    for _, v in pairs(game.Players:GetPlayers()) do
        if v ~= game.Players.LocalPlayer then 
            table.insert(pList, v.Name) 
        end
    end
    -- Menggunakan Refresh tanpa memicu penutupan otomatis jika memungkinkan di Rayfield
    TeleportDropdown:Refresh(pList, true) 
end

task.spawn(function()
   while true do
       UpdatePlayerList()
       task.wait(10) -- Jeda lebih lama agar daftar tidak sering berkedip/hilang
   end
end)
