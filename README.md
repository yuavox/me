-- lib v1.4-fix: colorpicker cursor offsets 0 + hsv bugs + animation mode fix
local uis = game:GetService("UserInputService")
local players = game:GetService("Players")
local ws = game:GetService("Workspace")
local http_service = game:GetService("HttpService")
local gui_service = game:GetService("GuiService")
local lighting = game:GetService("Lighting")
local run = game:GetService("RunService")
local stats = game:GetService("Stats")
local coregui = game:GetService("CoreGui")
local debris = game:GetService("Debris")
local tween_service = game:GetService("TweenService")
local rs = game:GetService("ReplicatedStorage")

local vec2 = Vector2.new
local vec3 = Vector3.new
local dim2 = UDim2.new
local dim = UDim.new
local rect = Rect.new
local cfr = CFrame.new

local color = Color3.new
local rgb = Color3.fromRGB
local hex = Color3.fromHex
local hsv = Color3.fromHSV
local rgbseq = ColorSequence.new
local rgbkey = ColorSequenceKeypoint.new

local camera = ws.CurrentCamera
local lp = players.LocalPlayer
local mouse = lp:GetMouse()
local gui_offset = gui_service:GetGuiInset().Y

local max = math.max
local floor = math.floor
local min = math.min
local abs = math.abs

if getgenv().library then
        getgenv().library:unload()
end

getgenv().library = {
        flags = {},
        config_flags = {},
        connections = {},
        notifications = {},
        instances = {},
        unload_callbacks = {},
        main_frame = {},
        config_holder,
        current_tab,
        current_element_open,
        dock_button_holder,
        gui,
        sin = 0,
        keybind_path,
        panel_open = false,

        directory = "invera.priv",
        folders = {
                "/fonts",
                "/configs",
        },
        font,
}

local flags = library.flags
local config_flags = library.config_flags
local _teGui = nil

local themes = {
        preset = {
                ["outline"] = rgb(56, 56, 56),
                ["inline"] = rgb(26, 26, 26),
                ["accent"] = rgb(255, 255, 255),
                ["contrast"] = rgb(8, 8, 8),
                ["bg"] = rgb(22, 22, 22),
                ["text"] = rgb(170, 170, 170),
                ["unselected_text"] = rgb(90, 90, 90),
                ["text_outline"] = rgb(0, 0, 0),
                ["glow"] = rgb(255, 255, 255),
        },

        utility = {
                ["outline"] = {
                        ["BackgroundColor3"] = {},
                        ["Color"] = {},
                },
                ["inline"] = {
                        ["BackgroundColor3"] = {},
                },
                ["accent"] = {
                        ["BackgroundColor3"] = {},
                        ["TextColor3"] = {},
                        ["ImageColor3"] = {},
                        ["BorderColor3"] = {},
                        ["ScrollBarImageColor3"] = {},
                },
                ["contrast"] = {
                        ["Color"] = {},
                        ["BackgroundColor3"] = {},
                },
                ["bg"] = {
                        ["BackgroundColor3"] = {},
                },
                ["text"] = {
                        ["TextColor3"] = {},
                },
                ["text_outline"] = {
                        ["Color"] = {},
                        ["TextStrokeColor3"] = {},
                },
                ["glow"] = {
                        ["ImageColor3"] = {},
                },
        },
}

local keys = {
        [Enum.KeyCode.LeftShift] = "LS",
        [Enum.KeyCode.RightShift] = "RS",
        [Enum.KeyCode.LeftControl] = "LC",
        [Enum.KeyCode.RightControl] = "RC",
        [Enum.KeyCode.Insert] = "INS",
        [Enum.KeyCode.Backspace] = "BS",
        [Enum.KeyCode.Return] = "Ent",
        [Enum.KeyCode.LeftAlt] = "LA",
        [Enum.KeyCode.RightAlt] = "RA",
        [Enum.KeyCode.CapsLock] = "CAPS",
        [Enum.KeyCode.One] = "1",
        [Enum.KeyCode.Two] = "2",
        [Enum.KeyCode.Three] = "3",
        [Enum.KeyCode.Four] = "4",
        [Enum.KeyCode.Five] = "5",
        [Enum.KeyCode.Six] = "6",
        [Enum.KeyCode.Seven] = "7",
        [Enum.KeyCode.Eight] = "8",
        [Enum.KeyCode.Nine] = "9",
        [Enum.KeyCode.Zero] = "0",
        [Enum.KeyCode.KeypadOne] = "Num1",
        [Enum.KeyCode.KeypadTwo] = "Num2",
        [Enum.KeyCode.KeypadThree] = "Num3",
        [Enum.KeyCode.KeypadFour] = "Num4",
        [Enum.KeyCode.KeypadFive] = "Num5",
        [Enum.KeyCode.KeypadSix] = "Num6",
        [Enum.KeyCode.KeypadSeven] = "Num7",
        [Enum.KeyCode.KeypadEight] = "Num8",
        [Enum.KeyCode.KeypadNine] = "Num9",
        [Enum.KeyCode.KeypadZero] = "Num0",
        [Enum.KeyCode.Minus] = "-",
        [Enum.KeyCode.Equals] = "=",
        [Enum.KeyCode.Tilde] = "~",
        [Enum.KeyCode.LeftBracket] = "[",
        [Enum.KeyCode.RightBracket] = "]",
        [Enum.KeyCode.RightParenthesis] = ")",
        [Enum.KeyCode.LeftParenthesis] = "(",
        [Enum.KeyCode.Semicolon] = ",",
        [Enum.KeyCode.Quote] = "'",
        [Enum.KeyCode.BackSlash] = "\\",
        [Enum.KeyCode.Comma] = ",",
        [Enum.KeyCode.Period] = ".",
        [Enum.KeyCode.Slash] = "/",
        [Enum.KeyCode.Asterisk] = "*",
        [Enum.KeyCode.Plus] = "+",
        [Enum.KeyCode.Period] = ".",
        [Enum.KeyCode.Backquote] = "`",
        [Enum.UserInputType.MouseButton1] = "MB1",
        [Enum.UserInputType.MouseButton2] = "MB2",
        [Enum.UserInputType.MouseButton3] = "MB3",
        [Enum.KeyCode.Escape] = "ESC",
        [Enum.KeyCode.Space] = "SPC",
}

library.__index = library

for _, path in next, library.folders do
        makefolder(library.directory .. path)
end

if not isfile(library.directory .. "/fonts/main.ttf") then
        writefile(
                library.directory .. "/fonts/main.ttf",
                game:HttpGet("https://github.com/i77lhm/storage/raw/refs/heads/main/fonts/fs-tahoma-8px.ttf")
        )
end

local tahoma = {
        name = "SmallestPixel7",
        faces = {
                {
                        name = "Regular",
                        weight = 400,
                        style = "normal",
                        assetId = getcustomasset(library.directory .. "/fonts/main.ttf"),
                },
        },
}

if not isfile(library.directory .. "/fonts/main_encoded.ttf") then
        writefile(library.directory .. "/fonts/main_encoded.ttf", http_service:JSONEncode(tahoma))
end

library.font = Font.new(getcustomasset(library.directory .. "/fonts/main_encoded.ttf"), Enum.FontWeight.Regular)

function library.to_screen_point(position)
        return camera:WorldToViewportPoint(position)
end

function library:unload()
        for _, cb in ipairs(library.unload_callbacks or {}) do
                pcall(cb)
        end

        library.gui:Destroy()

        if _teGui then
                pcall(function() _teGui:Destroy() end)
                _teGui = nil
        end

        for _, connection in library.connections do
                connection:Disconnect()
        end

        for _, item in library.instances do
                item:Destroy()
        end

        getgenv().library = nil
end

function library:convert_string_rgb(str)
        local values = {}

        for value in string.gmatch(str, "[^,]+") do
                table.insert(values, tonumber(value))
        end

        if #values == 4 then
                local r, g, b, a = values[1], values[2], values[3], values[4]

                return r, g, b, a
        else
                library:notification({ text = "Input a correct RGBA value (in the format 255, 255, 255, 0.5)" })
        end
end

function library:connection(signal, callback)
        local connection = signal:Connect(callback)

        table.insert(library.connections, connection)

        return connection
end

function library:make_draggable(frame, drag_handler)
        local gui = frame
        local dragging = false
        local dragInput
        local dragStart
        local dragOrigin
        local dragTarget = gui.Position

        local handler = drag_handler or gui
        library:connection(handler.InputBegan, function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
                        dragging = true
                        dragStart = input.Position
                        dragOrigin = gui.Position
                        dragTarget = gui.Position

                        local connection
                        connection = input.Changed:Connect(function()
                                if input.UserInputState == Enum.UserInputState.End then
                                        dragging = false
                                        connection:Disconnect()
                                end
                        end)
                end
        end)

        library:connection(handler.InputChanged, function(input)
                if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
                        dragInput = input
                end
        end)

        library:connection(uis.InputChanged, function(input)
                if dragging and input == dragInput then
                        local delta = input.Position - dragStart
                        dragTarget = UDim2.new(dragOrigin.X.Scale, dragOrigin.X.Offset + delta.X, dragOrigin.Y.Scale, dragOrigin.Y.Offset + delta.Y)
                end
        end)

        library:connection(game:GetService("RunService").Heartbeat, function(dt)
                if dragging or (gui.Position.X.Offset ~= dragTarget.X.Offset or gui.Position.Y.Offset ~= dragTarget.Y.Offset) then
                        gui.Position = gui.Position:Lerp(dragTarget, math.clamp(dt * 15, 0, 1))
                end
        end)
end

function library:make_resizable(frame)
        local Frame = Instance.new("TextButton")
        Frame.Position = dim2(1, -10, 1, -10)
        Frame.BorderColor3 = rgb(0, 0, 0)
        Frame.Size = dim2(0, 10, 0, 10)
        Frame.BorderSizePixel = 0
        Frame.BackgroundColor3 = rgb(255, 255, 255)
        Frame.Parent = frame
        Frame.BackgroundTransparency = 1
        Frame.Text = ""

        local resizing = false
        local start_size
        local start
        local og_size = frame.Size

        Frame.InputBegan:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 then
                        resizing = true
                        start = input.Position
                        start_size = frame.Size
                end
        end)

        Frame.InputEnded:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 then
                        resizing = false
                end
        end)

        library:connection(uis.InputChanged, function(input, game_event)
                if resizing and input.UserInputType == Enum.UserInputType.MouseMovement then
                        local mouse_pos = vec2(mouse.X, mouse.Y)
                        local viewport_x = camera.ViewportSize.X
                        local viewport_y = camera.ViewportSize.Y

                        current_size = dim2(
                                start_size.X.Scale,
                                math.clamp(start_size.X.Offset + (input.Position.X - start.X), og_size.X.Offset, viewport_x),
                                start_size.Y.Scale,
                                math.clamp(start_size.Y.Offset + (input.Position.Y - start.Y), og_size.Y.Offset, viewport_y)
                        )
                        frame.Size = current_size
                end
        end)
end

function library:new_item(class, properties)
        local ins = Instance.new(class)

        for _, v in next, properties do
                ins[_] = v
        end

        table.insert(library.instances, ins)

        return ins
end

function library:animation(text)
        local pattern = {}
        for i = 1, tonumber(text:len()) do
                table.insert(pattern, string.sub(text, 1, i))
        end
        for i = tonumber(text:len()) - 1, 0, -1 do
                table.insert(pattern, string.sub(text, 1, i))
        end
        return pattern
end

function library:convert_enum(enum)
        local enum_parts = {}

        for part in string.gmatch(enum, "[%w_]+") do
                table.insert(enum_parts, part)
        end

        local enum_table = Enum
        for i = 2, #enum_parts do
                local enum_item = enum_table[enum_parts[i]]

                enum_table = enum_item
        end

        return enum_table
end

function library:config_list_update()
        if not library.config_holder then
                return
        end

        local list = {}

        for idx, file in next, listfiles(library.directory .. "/configs") do
                local name = file.split(file, "/configs/")[2]
                name = name.split(name, ".cfg")[1]
                list[#list + 1] = name
        end

        library.config_holder:refresh_options(list)
end

function library:get_config()
        local Config = {}

        for _, v in flags do
                if type(v) == "table" and v.key then
                        Config[_] = { active = v.active, mode = v.mode, key = tostring(v.key) }
                elseif type(v) == "table" and v["Transparency"] and v["Color"] then
                        Config[_] = { Transparency = v["Transparency"], Color = v["Color"]:ToHex() }
                else
                        Config[_] = v
                end
        end

        return http_service:JSONEncode(Config)
end

function library:load_config(config_json)
        local config = http_service:JSONDecode(config_json)

        for _, v in next, config do
                local function_set = library.config_flags[_]

                if function_set then
                        if type(v) == "table" and v["Transparency"] and v["Color"] then
                                function_set(hex(v["Color"]), v["Transparency"])
                        elseif type(v) == "table" and v["active"] then
                                function_set(v)
                        else
                                function_set(v)
                        end
                end
        end
end

function library:round(number, float)
        local multiplier = 1 / (float or 1)
        return math.floor(number * multiplier + 0.5) / multiplier
end

function library:apply_theme(instance, theme, property)
        if not themes.utility[theme] then return end
        if not themes.utility[theme][property] then
                themes.utility[theme][property] = {}
        end
        table.insert(themes.utility[theme][property], instance)
end

function library:update_theme(theme, color)
        if not themes.utility[theme] then return end

        for propName, instances in next, themes.utility[theme] do
                for _, object in next, instances do
                        pcall(function() object[propName] = color end)
                end
        end

        themes.preset[theme] = color
end

function library:connection(signal, callback)
        local connection = signal:Connect(callback)

        table.insert(library.connections, connection)

        return connection
end

function library:create(instance, options)
        local ins = Instance.new(instance)

        for prop, value in next, options do
                ins[prop] = value
        end

        return ins
end

library.gui = library:create("ScreenGui", {
        Enabled = true,
        Parent = coregui,
        Name = "",
        DisplayOrder = 2,
        ZIndexBehavior = 1,
})

function library:window(properties)
        local cfg = {
                name = properties.Name or properties.name or properties.Title or properties.title or "sp4m.wtf",
                size = properties.Size or properties.size or dim2(0, 480, 0, 480),
        }

        local __holder = library:create("Frame", {
                Parent = library.gui,
                Name = "Watermark",
                Active = true,
                Position = UDim2.new(0, 20, 0, 20),
                BackgroundColor3 = themes.preset.bg,
                BorderSizePixel = 0,
                ZIndex = 1000,
                AutomaticSize = Enum.AutomaticSize.XY,
        })
        library:make_draggable(__holder)
        library:apply_theme(__holder, "bg", "BackgroundColor3")

        local wm_stroke = library:create("UIStroke", {
                Parent = __holder,
                Color = themes.preset.outline,
                Thickness = 1,
                LineJoinMode = Enum.LineJoinMode.Miter,
        })
        library:apply_theme(wm_stroke, "outline", "Color")

        local accent_line = library:create("Frame", {
                Parent = __holder,
                Name = "",
                Size = UDim2.new(1, 0, 0, 2),
                BorderSizePixel = 0,
                BackgroundColor3 = themes.preset.accent,
        })
        library:apply_theme(accent_line, "accent", "BackgroundColor3")

        local glow = library:create("ImageLabel", {
                Parent = __holder,
                Name = "",
                ImageColor3 = themes.preset.accent,
                ScaleType = Enum.ScaleType.Slice,
                ImageTransparency = 0.85,
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                Image = "http://www.roblox.com/asset/?id=18245826428",
                BackgroundTransparency = 1,
                Position = UDim2.new(0, -20, 0, -20),
                Size = UDim2.new(1, 40, 0, 42),
                ZIndex = 2,
                BorderSizePixel = 0,
                SliceCenter = Rect.new(Vector2.new(21, 21), Vector2.new(79, 79)),
        })
        library:apply_theme(glow, "accent", "ImageColor3")

        library:create("UIPadding", {
                Parent = __holder,
                PaddingTop = UDim.new(0, 6),
                PaddingBottom = UDim.new(0, 6),
                PaddingLeft = UDim.new(0, 10),
                PaddingRight = UDim.new(0, 10),
        })

        local name = library:create("TextLabel", {
                Parent = __holder,
                Name = "",
                FontFace = library.font,
                TextColor3 = Color3.fromRGB(255, 255, 255),
                Text = "invera.priv ~ Fps: 0 ~ Ping: 0 ~ [game]",
                TextStrokeTransparency = 0,
                TextStrokeColor3 = Color3.fromRGB(0, 0, 0),
                BackgroundTransparency = 1,
                BorderSizePixel = 0,
                TextSize = 13,
                AutomaticSize = Enum.AutomaticSize.XY,
        })
        library:apply_theme(name, "accent", "TextColor3")

        local run = game:GetService("RunService")
        local stats = game:GetService("Stats")
        local mps = game:GetService("MarketplaceService")

        local fps = 0
        local frames = 0
        local ping = 0

        local game_name = "Roblox"
        pcall(function()
                game_name = mps:GetProductInfo(game.PlaceId).Name
        end)

        run.Heartbeat:Connect(function()
                frames = frames + 1
        end)

        task.spawn(function()
                while true do
                        task.wait(1)
                        fps = frames
                        frames = 0
                        
                        pcall(function()
                                ping = math.round(stats.Network.ServerStatsItem["Data Ping"]:GetValue())
                        end)

                        if __holder.Visible then
                                name.Text = string.format("invera.priv ~ Fps: %d ~ Ping: %d ~ [%s]", fps, ping, game_name)
                        end
                end
        end)

        local inline1 = library:create("Frame", {
                Parent = library.gui,
                Name = "",
                Active = true,
                Position = UDim2.new(0.5, -cfg.size.X.Offset / 2, 0.5, -cfg.size.Y.Offset / 2),
                BorderColor3 = Color3.fromRGB(8, 8, 8),
                ZIndex = 2,
                Size = cfg.size,
                BackgroundColor3 = Color3.fromRGB(40, 40, 40),
        })
        table.insert(library.main_frame, inline1)
        library:apply_theme(inline1, "inline", "BackgroundColor3")
        library:apply_theme(inline1, "outline", "BorderColor3")
        local WINDOW_PATH = inline1
        library:make_draggable(inline1)
        library:make_resizable(inline1)

        local inline2 = library:create("Frame", {
                Parent = inline1,
                Name = "",
                Position = UDim2.new(0, 2, 0, 2),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Size = UDim2.new(1, -4, 1, -4),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(26, 26, 26),
        })
        library:apply_theme(inline2, "inline", "BackgroundColor3")

        local main = library:create("Frame", {
                Parent = inline2,
                Name = "",
                Position = UDim2.new(0, 2, 0, 2),
                BorderColor3 = Color3.fromRGB(57, 57, 57),
                Size = UDim2.new(1, -4, 1, -4),
                BackgroundColor3 = Color3.fromRGB(26, 26, 26),
        })
        library:apply_theme(main, "inline", "BackgroundColor3")
        library:apply_theme(main, "outline", "BorderColor3")

        local tab_buttons = library:create("Frame", {
                Parent = main,
                Name = "",
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 16, 0, 4),
                Size = UDim2.new(1, -32, 0, 0),
                ZIndex = 2,
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        cfg["tab_holder"] = tab_buttons

        local list = library:create("UIListLayout", {
                Parent = tab_buttons,
                Name = "",
                FillDirection = Enum.FillDirection.Horizontal,
                HorizontalAlignment = Enum.HorizontalAlignment.Center,
                HorizontalFlex = Enum.UIFlexAlignment.Fill,
                Padding = UDim.new(0, 6),
                SortOrder = Enum.SortOrder.LayoutOrder,
        })

        local tab_inline = library:create("Frame", {
                Parent = main,
                Name = "",
                Position = UDim2.new(0, 15, 0, 33),
                BorderColor3 = Color3.fromRGB(19, 19, 19),
                Size = UDim2.new(1, -30, 1, -48),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(19, 19, 19),
        })
        library:apply_theme(tab_inline, "contrast", "BackgroundColor3")

        local tabs = library:create("Frame", {
                Parent = tab_inline,
                Name = "",
                Position = UDim2.new(0, 2, 0, 2),
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Size = UDim2.new(1, -4, 1, -4),
                BackgroundColor3 = themes.preset.bg,
        })
        cfg["tab_instance_holder"] = tabs
        library:apply_theme(tabs, "bg", "BackgroundColor3")

        local accent_line = library:create("Frame", {
                Parent = inline1,
                Name = "",
                BorderColor3 = Color3.fromRGB(34, 34, 34),
                Size = UDim2.new(1, 0, 0, 2),
                BorderSizePixel = 0,
                BackgroundColor3 = themes.preset.accent,
        })

        library:apply_theme(accent_line, "accent", "BackgroundColor3")

        local name = library:create("TextLabel", {
                Parent = inline1,
                Name = "",
                FontFace = library.font,
                TextColor3 = Color3.fromRGB(170, 170, 170),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Text = cfg.name,
                TextStrokeTransparency = 0.5,
                BorderSizePixel = 0,
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 0, 0, -1),
                Size = UDim2.new(1, 0, 0, 1),
                ZIndex = 2,
                TextSize = 12,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })
        library:apply_theme(name, "text", "TextColor3")

        local glow = library:create("ImageLabel", {
                Parent = inline1,
                Name = "",
                ImageColor3 = themes.preset.accent,
                ScaleType = Enum.ScaleType.Slice,
                ImageTransparency = 0.8999999761581421,
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                Image = "http://www.roblox.com/asset/?id=18245826428",
                BackgroundTransparency = 1,
                Position = UDim2.new(0, -20, 0, -20),
                Size = UDim2.new(1, 40, 0, 42),
                ZIndex = 2,
                BorderSizePixel = 0,
                SliceCenter = Rect.new(Vector2.new(21, 21), Vector2.new(79, 79)),
        })
        library:apply_theme(glow, "accent", "ImageColor3")

        local depth = library:create("Frame", {
                Parent = inline1,
                Name = "",
                BackgroundTransparency = 0.5,
                Position = UDim2.new(0, 0, 0, 1),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Size = UDim2.new(1, 0, 0, 1),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(0, 0, 0),
        })

        local holder = library:create("Frame", {
                Parent = inline1,
                Name = "",
                BackgroundTransparency = 1,
                Position = UDim2.new(1, 20, 0, 0),
                BorderColor3 = Color3.fromRGB(19, 19, 19),
                ZIndex = 2,
                AutomaticSize = Enum.AutomaticSize.XY,
                BackgroundColor3 = Color3.fromRGB(40, 40, 40),
        })

        task.spawn(function()
                while true do
                        local speed = flags["color_picker_anim_speed"]
                        if speed and speed > 0 and TEXT_ANIMATION_GRADIENT and TEXT_ANIMATION_GRADIENT.Parent then
                                library.sin = math.abs(math.sin(tick() * speed))
                                TEXT_ANIMATION_GRADIENT.Color = ColorSequence.new({
                                        ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 255)),
                                        ColorSequenceKeypoint.new(math.abs(math.sin(tick())), themes.preset.accent),
                                        ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 255, 255)),
                                })
                        end
                        task.wait(0.1)
                end
        end)

        local esp_preview = library:create("Frame", {
                Parent = library.gui,
                Name = "",
                Visible = false,
                Active = true,
                Position = UDim2.new(
                        0,
                        inline1.AbsolutePosition.X + inline1.AbsoluteSize.X + 8,
                        0,
                        inline1.AbsolutePosition.Y + 1
                ),
                BorderColor3 = Color3.fromRGB(8, 8, 8),
                Size = UDim2.new(0, 328, 0, 376),
                BackgroundColor3 = Color3.fromRGB(56, 56, 56),
        })
        library:make_draggable(esp_preview)
        library:make_resizable(esp_preview)

        local name = library:create("TextLabel", {
                Parent = esp_preview,
                Name = "",
                FontFace = library.font,
                TextColor3 = Color3.fromRGB(170, 170, 170),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Text = "esp preview",
                TextStrokeTransparency = 0.5,
                BorderSizePixel = 0,
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 0, 0, -1),
                Size = UDim2.new(1, 0, 0, 1),
                ZIndex = 2,
                TextSize = 12,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local UIPadding = library:create("UIPadding", {
                Parent = name,
                Name = "",
        })

        local main = library:create("Frame", {
                Parent = esp_preview,
                Name = "",
                Position = UDim2.new(0, 4, 0, 4),
                BorderColor3 = Color3.fromRGB(26, 26, 26),
                Size = UDim2.new(1, -8, 1, -8),
                BorderSizePixel = 2,
                BackgroundColor3 = Color3.fromRGB(26, 26, 26),
        })

        library:create("UIStroke", {
                Parent = main,
                Name = "",
                Color = Color3.fromRGB(57, 57, 57),
                LineJoinMode = Enum.LineJoinMode.Miter,
        })

        local tabs = library:create("Frame", {
                Parent = main,
                Name = "",
                Position = UDim2.new(0, 8, 0, 8),
                BorderColor3 = Color3.fromRGB(8, 8, 8),
                Size = UDim2.new(1, -16, 1, -16),
                BorderSizePixel = 2,
                BackgroundColor3 = themes.preset.bg,
        })
        library:apply_theme(tabs, "bg", "BackgroundColor3")

        library:create("UIStroke", {
                Parent = tabs,
                Name = "",
                Color = Color3.fromRGB(57, 57, 57),
                LineJoinMode = Enum.LineJoinMode.Miter,
        })

        local hitpart = library:create("Frame", {
                Parent = tabs,
                Name = "",
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 2, 0, 20),
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Size = UDim2.new(1, -4, 1, -4),
                BackgroundColor3 = themes.preset.bg,
        })
        library:apply_theme(hitpart, "bg", "BackgroundColor3")

        local head = library:create("Frame", {
                Parent = hitpart,
                Name = "",
                Position = UDim2.new(0.5, -25, 0, 16),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Size = UDim2.new(0, 50, 0, 44),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(38, 38, 38),
        })

        local torso = library:create("Frame", {
                Parent = hitpart,
                Name = "",
                Position = UDim2.new(0.5, -42, 0, 64),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Size = UDim2.new(0, 84, 0, 90),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(38, 38, 38),
        })

        local l_arm = library:create("Frame", {
                Parent = hitpart,
                Name = "",
                Position = UDim2.new(0.5, -86, 0, 64),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Size = UDim2.new(0, 40, 0, 90),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(38, 38, 38),
        })

        local r_arm = library:create("Frame", {
                Parent = hitpart,
                Name = "",
                Position = UDim2.new(0.5, 46, 0, 64),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Size = UDim2.new(0, 40, 0, 90),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(38, 38, 38),
        })

        local r_leg = library:create("Frame", {
                Parent = hitpart,
                Name = "",
                Position = UDim2.new(0.5, 2, 0, 158),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Size = UDim2.new(0, 40, 0, 90),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(38, 38, 38),
        })

        local l_leg = library:create("Frame", {
                Parent = hitpart,
                Name = "",
                Position = UDim2.new(0.5, -42, 0, 158),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Size = UDim2.new(0, 40, 0, 90),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(38, 38, 38),
        })

        local hrp = library:create("Frame", {
                Parent = hrp_out,
                Name = "",
                Position = UDim2.new(0, 4, 0, 4),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Size = UDim2.new(1, -8, 1, -8),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(38, 38, 38),
        })

        local glow_patterns = {}

        for _, v in next, hitpart:GetChildren() do
                local glow = library:create("ImageLabel", {
                        Parent = v,
                        Name = "",
                        Visible = false,
                        ImageColor3 = themes.preset.accent,
                        ScaleType = Enum.ScaleType.Slice,
                        ImageTransparency = 0.8999999761581421,
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                        Image = "http://www.roblox.com/asset/?id=18245826428",
                        BackgroundTransparency = 1,
                        Position = UDim2.new(0, -20, 0, -20),
                        Size = UDim2.new(1, 40, 1, 40),
                        ZIndex = 2,
                        BorderSizePixel = 0,
                        SliceCenter = Rect.new(Vector2.new(21, 21), Vector2.new(79, 79)),
                })

                library:apply_theme(glow, "accent", "ImageColor3")

                table.insert(glow_patterns, glow)
        end

        function cfg.preview_chams(bool)
                for _, glow in next, glow_patterns do
                        glow.Visible = bool
                end

                for _, part in next, hitpart:GetChildren() do
                        part.BackgroundColor3 = bool and themes.preset.accent or Color3.fromRGB(38, 38, 38)
                end
        end

        local player = library:create("Frame", {
                Parent = tabs,
                Name = "",
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 43, 0, 28),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Size = UDim2.new(1, -86, 1, -106),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local line_holder = library:create("Frame", {
                Parent = player,
                Name = "",
                Size = UDim2.new(1, 0, 1, 0),
                ZIndex = 50,
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local box_outline = library:create("Frame", {
                Parent = line_holder,
                Name = "",
                BackgroundTransparency = 1,
                Position = UDim2.new(0, -1, 0, -1),
                ZIndex = 50,
                Size = UDim2.new(1, 2, 1, 2),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local BoxLine2 = library:create("UIStroke", {
                Parent = box_outline,
                Name = "",
                LineJoinMode = Enum.LineJoinMode.Miter,
        })

        local box_color = library:create("UIStroke", {
                Parent = line_holder,
                Name = "",
                Color = Color3.fromRGB(255, 255, 255),
                LineJoinMode = Enum.LineJoinMode.Miter,
        })

        local corner_box = library:create("Frame", {
                Parent = line_holder,
                Name = "",
                Visible = false,
                BackgroundTransparency = 1,
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Size = UDim2.new(1, 0, 1, 0),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local top_left = library:create("Frame", {
                Parent = corner_box,
                Name = "",
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                ZIndex = 50,
                Size = UDim2.new(0, 1, 0.30000001192092896, 0),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local top_right = library:create("Frame", {
                Parent = corner_box,
                Name = "",
                AnchorPoint = Vector2.new(1, 0),
                Position = UDim2.new(1, -1, 0, 0),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                ZIndex = 50,
                Size = UDim2.new(0, 1, 0.30000001192092896, 0),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local bottom_left = library:create("Frame", {
                Parent = corner_box,
                Name = "",
                AnchorPoint = Vector2.new(0, 1),
                Position = UDim2.new(0, 0, 1, 0),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                ZIndex = 50,
                Size = UDim2.new(0.4000000059604645, 0, 0, 1),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local bottom_right = library:create("Frame", {
                Parent = corner_box,
                Name = "",
                AnchorPoint = Vector2.new(1, 1),
                Position = UDim2.new(1, -1, 1, 0),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                ZIndex = 50,
                Size = UDim2.new(0.4000000059604645, 0, 0, 1),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local bottom_left2 = library:create("Frame", {
                Parent = corner_box,
                Name = "",
                AnchorPoint = Vector2.new(0, 1),
                Position = UDim2.new(0, 0, 1, 0),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                ZIndex = 50,
                Size = UDim2.new(0, 1, 0.30000001192092896, 0),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local bottom_right2 = library:create("Frame", {
                Parent = corner_box,
                Name = "",
                AnchorPoint = Vector2.new(1, 1),
                Position = UDim2.new(1, 0, 1, 0),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                ZIndex = 50,
                Size = UDim2.new(0, 1, 0.30000001192092896, 0),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local top_left2 = library:create("Frame", {
                Parent = corner_box,
                Name = "",
                AnchorPoint = Vector2.new(0, 1),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                ZIndex = 50,
                Size = UDim2.new(0.4000000059604645, 0, 0, 1),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local top_right2 = library:create("Frame", {
                Parent = corner_box,
                Name = "",
                AnchorPoint = Vector2.new(1, 1),
                Position = UDim2.new(1, -1, 0, 0),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                ZIndex = 50,
                Size = UDim2.new(0.4000000059604645, 0, 0, 1),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        function cfg.preview_corner_boxes(bool)
                corner_box.Visible = bool == "Corner" and true or false
                BoxLine2.Enabled = bool == "Corner" and false or true
                box_outline.Visible = bool == "Corner" and false or true
                box_color.Enabled = bool == "Corner" and false or true
        end

        function cfg.preview_bounding_box(bool)
                BoxLine2.Enabled = bool
                box_outline.Visible = bool
                box_color.Enabled = bool
        end

        local bottom_holder = library:create("Frame", {
                Parent = line_holder,
                Name = "",
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                BackgroundTransparency = 1,
                Position = UDim2.new(0, -1, 1, 3),
                Size = UDim2.new(1, 2, 0, 0),
                BorderSizePixel = 0,
                AutomaticSize = Enum.AutomaticSize.Y,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local UIListLayout = library:create("UIListLayout", {
                Parent = bottom_holder,
                Name = "",
                Padding = UDim.new(0, 2),
                SortOrder = Enum.SortOrder.LayoutOrder,
        })

        local UIPadding = library:create("UIPadding", {
                Parent = bottom_holder,
                Name = "",
                PaddingTop = UDim.new(0, 1),
        })

        local bar_holder = library:create("Frame", {
                Parent = bottom_holder,
                Name = "",
                BackgroundTransparency = 1,
                Size = UDim2.new(1, 0, 0, 4),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                BorderSizePixel = 0,
                AutomaticSize = Enum.AutomaticSize.Y,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local reload_bar = library:create("Frame", {
                Parent = bar_holder,
                Name = "",
                Size = UDim2.new(1, 0, 0, 4),
                ZIndex = 50,
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(0, 0, 0),
        })

        function cfg.preview_reload_bar(bool)
                bar_holder.Visible = bool
        end

        local reload_slider = library:create("Frame", {
                Parent = reload_bar,
                Name = "",
                Size = UDim2.new(0.5, -2, 0, 2),
                Position = UDim2.new(0, 1, 0, 1),
                ZIndex = 50,
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(28, 145, 255),
        })

        local gradient = library:create("UIGradient", {
                Parent = reload_slider,
                Name = "",
                Color = ColorSequence.new({
                        ColorSequenceKeypoint.new(0, Color3.fromRGB(0, 255, 238)),
                        ColorSequenceKeypoint.new(1, Color3.fromRGB(0, 255, 238)),
                }),
        })

        local UIStroke = library:create("UIStroke", {
                Parent = distance,
                Name = "",
                LineJoinMode = Enum.LineJoinMode.Miter,
        })

        local weapon = library:create("TextLabel", {
                Parent = bottom_holder,
                Name = "",
                FontFace = library.font,
                TextColor3 = Color3.fromRGB(255, 255, 255),
                Text = "double barrel",
                TextStrokeTransparency = 0,
                AnchorPoint = Vector2.new(0.5, 0.5),
                BackgroundTransparency = 1,
                Position = UDim2.new(0.5, 0, 0.031031031161546707, 0),
                AutomaticSize = Enum.AutomaticSize.Y,
                ZIndex = 50,
                TextSize = 12,
                Size = UDim2.new(1, 0, 0, 4),
        })

        function cfg.preview_weapon(bool)
                weapon.Visible = bool
        end

        local UIStroke = library:create("UIStroke", {
                Parent = weapon,
                Name = "",
                LineJoinMode = Enum.LineJoinMode.Miter,
        })

        local UIStroke = library:create("UIStroke", {
                Parent = distance,
                Name = "",
                LineJoinMode = Enum.LineJoinMode.Miter,
        })

        local image_holder = library:create("Frame", {
                Parent = bottom_holder,
                Name = "",
                BackgroundTransparency = 1,
                Size = UDim2.new(1, 0, 0, 0),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                BorderSizePixel = 0,
                AutomaticSize = Enum.AutomaticSize.Y,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        function cfg.preview_icons(bool)
                image_holder.Visible = bool
        end

        local ImageLabel = library:create("ImageLabel", {
                Parent = image_holder,
                Name = "",
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                AnchorPoint = Vector2.new(0.5, 0),
                Image = "rbxassetid://130516018594923",
                BackgroundTransparency = 1,
                Position = UDim2.new(0.5, 0, 0, 0),
                Size = UDim2.new(0, 64, 0, 27),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local armor = library:create("Frame", {
                Parent = line_holder,
                Name = "",
                Position = UDim2.new(0, -14, 0, -2),
                Size = UDim2.new(0, 4, 1, 4),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(0, 0, 0),
        })

        function cfg.preview_armor(bool)
                armor.Visible = bool
        end

        local armor_slider = library:create("Frame", {
                Parent = armor,
                Name = "",
                Position = UDim2.new(0, 1, 0, 1),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Size = UDim2.new(0.5, 0, 1, -2),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(0, 13, 255),
        })

        local armor_gradient = library:create("UIGradient", {
                Parent = armor_slider,
                Name = "",
                Rotation = 90,
                Color = ColorSequence.new({
                        ColorSequenceKeypoint.new(0, Color3.fromRGB(0, 242, 255)),
                        ColorSequenceKeypoint.new(1, Color3.fromRGB(0, 17, 255)),
                }),
                Enabled = false,
        })

        local armor_text = library:create("TextLabel", {
                Parent = armor_slider,
                Name = "",
                ZIndex = 99,
                FontFace = library.font,
                TextColor3 = Color3.fromRGB(0, 13, 255),
                Text = "100",
                Position = UDim2.new(0, -2, 0.75, -2),
                TextStrokeTransparency = 0,
                AnchorPoint = Vector2.new(1, 0),
                BorderSizePixel = 0,
                BackgroundTransparency = 1,
                TextXAlignment = Enum.TextXAlignment.Right,
                Active = true,
                TextYAlignment = Enum.TextYAlignment.Top,
                TextSize = 12,
                BackgroundColor3 = Color3.fromRGB(26, 255, 0),
        })

        library:create("UIStroke", {
                Parent = armor_text,
                Name = "",
                LineJoinMode = Enum.LineJoinMode.Miter,
        })

        local health = library:create("Frame", {
                Parent = line_holder,
                Name = "",
                Position = UDim2.new(0, -8, 0, -2),
                Size = UDim2.new(0, 4, 1, 4),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(0, 0, 0),
        })

        function cfg.preview_health(bool)
                health.Visible = bool
        end

        local health_slider = library:create("Frame", {
                Parent = health,
                Name = "",
                Position = UDim2.new(0, 1, 0, 1),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Size = UDim2.new(0.5, 0, 1, -2),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(0, 255, 42),
        })

        local health_text = library:create("TextLabel", {
                Parent = health_slider,
                Name = "",
                Visible = false,
                FontFace = library.font,
                TextColor3 = Color3.fromRGB(0, 255, 0),
                Text = "100",
                ZIndex = 99,
                TextStrokeTransparency = 0,
                AnchorPoint = Vector2.new(0.5, 0.5),
                Position = UDim2.new(0, -4, 0.5, -2),
                BackgroundTransparency = 1,
                TextXAlignment = Enum.TextXAlignment.Right,
                Active = true,
                TextYAlignment = Enum.TextYAlignment.Top,
                TextSize = 12,
                BackgroundColor3 = Color3.fromRGB(26, 255, 0),
        })

        local UIStroke = library:create("UIStroke", {
                Parent = health_text,
                Name = "",
                LineJoinMode = Enum.LineJoinMode.Miter,
        })

        local box_inline = library:create("Frame", {
                Parent = line_holder,
                Name = "",
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 1, 0, 1),
                ZIndex = 50,
                Size = UDim2.new(1, -2, 1, -2),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local BoxLine3 = library:create("UIStroke", {
                Parent = box_inline,
                Name = "",
                LineJoinMode = Enum.LineJoinMode.Miter,
        })

        local gradient = library:create("UIGradient", {
                Parent = line_holder,
                Name = "",
                Rotation = -180,
                Transparency = NumberSequence.new({
                        NumberSequenceKeypoint.new(0, 0.5),
                        NumberSequenceKeypoint.new(1, 0.5),
                }),
                Color = ColorSequence.new({
                        ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 0, 0)),
                        ColorSequenceKeypoint.new(1, Color3.fromRGB(0, 0, 0)),
                }),
        })

        function cfg.preview_filler(bool)
                line_holder.BackgroundTransparency = bool and 0 or 1
                gradient.Enabled = bool
        end

        local top_holder = library:create("Frame", {
                Parent = line_holder,
                Name = "",
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                AnchorPoint = Vector2.new(0, 1),
                BackgroundTransparency = 1,
                Position = UDim2.new(0, -2, 0, -4),
                Size = UDim2.new(1, 4, 0, 0),
                BorderSizePixel = 0,
                AutomaticSize = Enum.AutomaticSize.Y,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        library:create("UIListLayout", {
                Parent = top_holder,
                Name = "",
                Padding = UDim.new(0, 4),
                SortOrder = Enum.SortOrder.LayoutOrder,
        })

        library:create("UIPadding", {
                Parent = top_holder,
                Name = "",
                PaddingTop = UDim.new(0, 1),
        })

        local player_name = library:create("TextLabel", {
                Parent = top_holder,
                Name = "",
                RichText = true,
                TextColor3 = Color3.fromRGB(255, 255, 255),
                Text = "hello there",
                FontFace = library.font,
                AnchorPoint = Vector2.new(0, 1),
                AutomaticSize = Enum.AutomaticSize.Y,
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 0, 0, -2),
                BorderSizePixel = 0,
                ZIndex = 50,
                TextSize = 12,
                Size = UDim2.new(1, 0, 0, 0),
        })

        function cfg.preview_names(bool)
                player_name.Visible = bool
        end

        local UIStroke = library:create("UIStroke", {
                Parent = player_name,
                Name = "",
                LineJoinMode = Enum.LineJoinMode.Miter,
        })

        local accent_line = library:create("Frame", {
                Parent = esp_preview,
                Name = "",
                BorderColor3 = Color3.fromRGB(34, 34, 34),
                Size = UDim2.new(1, 0, 0, 2),
                BorderSizePixel = 0,
                BackgroundColor3 = themes.preset.accent,
        })

        library:apply_theme(accent_line, "accent", "BackgroundColor3")

        local depth = library:create("Frame", {
                Parent = esp_preview,
                Name = "",
                BackgroundTransparency = 0.5,
                Position = UDim2.new(0, 0, 0, 1),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Size = UDim2.new(1, 0, 0, 1),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(0, 0, 0),
        })

        local glow = library:create("ImageLabel", {
                Parent = esp_preview,
                Name = "",
                ImageColor3 = themes.preset.accent,
                ScaleType = Enum.ScaleType.Slice,
                ImageTransparency = 0.8999999761581421,
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                Image = "http://www.roblox.com/asset/?id=18245826428",
                BackgroundTransparency = 1,
                Position = UDim2.new(0, -20, 0, -20),
                Size = UDim2.new(1, 40, 0, 42),
                ZIndex = 2,
                BorderSizePixel = 0,
                SliceCenter = Rect.new(Vector2.new(21, 21), Vector2.new(79, 79)),
        })

        library:apply_theme(glow, "accent", "ImageColor3")

        local camera = workspace.CurrentCamera
        local selected_button
        local selected_player
        local player_buttons = {}

        function library.get_priority(player)
                if player_buttons[player.Name] and player_buttons[player.Name].priority then
                        return player_buttons[player.Name].priority.Text
                end
                return "Neutral"
        end

        local playerlist = library:create("Frame", {
                Parent = library.gui,
                Name = "PlayerList",
                Active = true,
                AnchorPoint = Vector2.new(0, 0),
                Position = UDim2.new(0, inline1.AbsolutePosition.X - 258 - 8, 0, inline1.AbsolutePosition.Y + 1),
                BorderColor3 = Color3.fromRGB(8, 8, 8),
                Size = UDim2.new(0, 240, 0, 328),
                BackgroundColor3 = themes.preset.bg,
                BorderSizePixel = 0,
        })
        library:make_draggable(playerlist)
        library:apply_theme(playerlist, "bg", "BackgroundColor3")
        library:make_resizable(playerlist)

        table.insert(library.main_frame, playerlist)

        library:create("UIStroke", {
                Parent = playerlist,
                Color = Color3.fromRGB(57, 57, 57),
                Thickness = 1,
                LineJoinMode = Enum.LineJoinMode.Miter,
        })

        local accent_line = library:create("Frame", {
                Parent = playerlist,
                Name = "",
                Size = UDim2.new(1, 0, 0, 2),
                BorderSizePixel = 0,
                BackgroundColor3 = themes.preset.accent,
        })
        library:apply_theme(accent_line, "accent", "BackgroundColor3")

        local glow = library:create("ImageLabel", {
                Parent = playerlist,
                Name = "",
                ImageColor3 = themes.preset.accent,
                ScaleType = Enum.ScaleType.Slice,
                ImageTransparency = 0.85,
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                Image = "http://www.roblox.com/asset/?id=18245826428",
                BackgroundTransparency = 1,
                Position = UDim2.new(0, -20, 0, -20),
                Size = UDim2.new(1, 40, 0, 42),
                ZIndex = 2,
                BorderSizePixel = 0,
                SliceCenter = Rect.new(Vector2.new(21, 21), Vector2.new(79, 79)),
        })
        library:apply_theme(glow, "accent", "ImageColor3")

        local name = library:create("TextLabel", {
                Parent = playerlist,
                Name = "",
                FontFace = library.font,
                TextColor3 = Color3.fromRGB(255, 255, 255),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Text = "playerlist",
                TextStrokeTransparency = 0,
                TextStrokeColor3 = Color3.fromRGB(0, 0, 0),
                BorderSizePixel = 0,
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 10, 0, 6),
                Size = UDim2.new(1, -20, 0, 16),
                ZIndex = 2,
                TextSize = 12,
                TextXAlignment = Enum.TextXAlignment.Left,
        })

        local __ScrollingFrame = library:create("ScrollingFrame", {
                Parent = playerlist,
                ScrollBarImageColor3 = themes.preset.accent,
                Active = true,
                AutomaticCanvasSize = Enum.AutomaticSize.Y,
                ScrollBarThickness = 2,
                Position = UDim2.new(0, 8, 0, 28),
                Size = UDim2.new(1, -16, 1, -36),
                BackgroundTransparency = 1,
                BorderSizePixel = 0,
                CanvasSize = UDim2.new(0, 0, 0, 0),
        })
        library:apply_theme(__ScrollingFrame, "accent", "ScrollBarImageColor3")

        library:create("UIListLayout", {
                Parent = __ScrollingFrame,
                Padding = UDim.new(0, 4),
                SortOrder = Enum.SortOrder.LayoutOrder,
        })

        local function create_player(player)
                local TextButton = library:create("TextButton", {
                        Parent = __ScrollingFrame,
                        Name = player.Name,
                        FontFace = library.font,
                        TextColor3 = Color3.fromRGB(180, 180, 180),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Text = "",
                        BackgroundTransparency = 1,
                        Size = UDim2.new(1, 0, 0, 24),
                        BorderSizePixel = 0,
                        AutomaticSize = Enum.AutomaticSize.Y,
                        TextSize = 12,
                        BackgroundColor3 = Color3.fromRGB(30, 30, 30),
                })

                local row_stroke = library:create("UIStroke", {
                        Parent = TextButton,
                        Color = Color3.fromRGB(45, 45, 45),
                        Thickness = 1,
                        ApplyStrokeMode = Enum.ApplyStrokeMode.Border,
                })

                local header = library:create("Frame", {
                        Parent = TextButton,
                        Name = "Header",
                        Size = UDim2.new(1, 0, 0, 24),
                        BackgroundTransparency = 1,
                })

                local name_label = library:create("TextLabel", {
                        Parent = header,
                        FontFace = library.font,
                        TextColor3 = Color3.fromRGB(200, 200, 200),
                        Text = player.Name,
                        TextXAlignment = Enum.TextXAlignment.Left,
                        Position = UDim2.new(0, 8, 0, 0),
                        Size = UDim2.new(0.6, -8, 1, 0),
                        BackgroundTransparency = 1,
                        TextStrokeTransparency = 0.5,
                        TextSize = 12,
                })

                local priority_label = library:create("TextLabel", {
                        Parent = header,
                        FontFace = library.font,
                        TextColor3 = Color3.fromRGB(150, 150, 150),
                        Text = "Neutral",
                        TextXAlignment = Enum.TextXAlignment.Right,
                        Position = UDim2.new(0.6, 0, 0, 0),
                        Size = UDim2.new(0.4, -8, 1, 0),
                        BackgroundTransparency = 1,
                        TextStrokeTransparency = 0.5,
                        TextSize = 12,
                })

                player_buttons[player.Name] = {
                        instance = TextButton,
                        priority = priority_label,
                }

                local left_accent = library:create("Frame", {
                        Parent = TextButton,
                        Name = "LeftAccent",
                        Size = UDim2.new(0, 2, 0, 24),
                        Position = UDim2.new(0, 0, 0, 0),
                        BorderSizePixel = 0,
                        BackgroundColor3 = Color3.fromRGB(45, 45, 45),
                })

                local expanded = library:create("Frame", {
                        Parent = TextButton,
                        Name = "Expanded",
                        Position = UDim2.new(0, 0, 0, 24),
                        Size = UDim2.new(1, 0, 0, 108),
                        BackgroundTransparency = 1,
                        Visible = false,
                        ClipsDescendants = true,
                })

                library:create("Frame", {
                        Parent = expanded,
                        Position = UDim2.new(0, 4, 0, 0),
                        Size = UDim2.new(1, -8, 0, 1),
                        BackgroundColor3 = Color3.fromRGB(45, 45, 45),
                        BorderSizePixel = 0,
                })

                local avatar_img = "rbxassetid://0"
                pcall(function()
                        avatar_img = players:GetUserThumbnailAsync(player.UserId, Enum.ThumbnailType.HeadShot, Enum.ThumbnailSize.Size48x48)
                end)

                local avatar = library:create("ImageLabel", {
                        Parent = expanded,
                        Size = UDim2.new(0, 48, 0, 48),
                        Position = UDim2.new(0, 8, 0, 8),
                        BackgroundColor3 = Color3.fromRGB(20, 20, 20),
                        BorderSizePixel = 0,
                        Image = avatar_img,
                })

                library:create("UIStroke", {
                        Parent = avatar,
                        Color = Color3.fromRGB(57, 57, 57),
                        Thickness = 1,
                })

                local function makeMiniButton(name, text, xPos, colorFn)
                        local wrap = library:create("Frame", {
                                Parent = expanded,
                                Size = UDim2.new(0, 44, 0, 18),
                                Position = UDim2.new(0, xPos, 0, 60),
                                BackgroundColor3 = Color3.fromRGB(8, 8, 8),
                                BorderSizePixel = 0,
                        })
                        local lbl = library:create("TextLabel", {
                                Parent = wrap,
                                Position = UDim2.new(0, 2, 0, 2),
                                Size = UDim2.new(1, -4, 1, -4),
                                BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                                BorderColor3 = Color3.fromRGB(56, 56, 56),
                                TextColor3 = Color3.fromRGB(170, 170, 170),
                                Text = text,
                                FontFace = library.font,
                                TextSize = 11,
                                TextStrokeTransparency = 0.5,
                                TextStrokeColor3 = Color3.fromRGB(0, 0, 0),
                        })
                        local btn = library:create("TextButton", {
                                Parent = wrap,
                                Size = UDim2.new(1, 0, 1, 0),
                                BackgroundTransparency = 1,
                                Text = "",
                                ZIndex = 4,
                        })
                        library:connection(btn.MouseEnter, function() lbl.TextColor3 = themes.preset.accent end)
                        library:connection(btn.MouseLeave, function() lbl.TextColor3 = Color3.fromRGB(170, 170, 170) end)
                        btn.MouseButton1Click:Connect(function()
                                priority_label.Text = name
                                priority_label.TextColor3 = colorFn()
                        end)
                        return wrap
                end

                makeMiniButton("Neutral", "Neutral", 8, function() return Color3.fromRGB(150, 150, 150) end)
                makeMiniButton("Friendly", "Friendly", 54, function() return themes.preset.accent end)
                makeMiniButton("Enemy", "Enemy", 100, function() return Color3.fromRGB(200, 80, 80) end)

                local actions = library:create("Frame", {
                        Parent = expanded,
                        Position = UDim2.new(0, 68, 0, 8),
                        Size = UDim2.new(1, -76, 0, 96),
                        BackgroundTransparency = 1,
                })

                library:create("UIListLayout", {
                        Parent = actions,
                        Padding = UDim.new(0, 4),
                        SortOrder = Enum.SortOrder.LayoutOrder,
                })

                local tp_wrap = library:create("Frame", {
                        Parent = actions,
                        Size = UDim2.new(1, 0, 0, 16),
                        BackgroundColor3 = Color3.fromRGB(8, 8, 8),
                        BorderSizePixel = 0,
                })
                local tp_lbl = library:create("TextLabel", {
                        Parent = tp_wrap,
                        Position = UDim2.new(0, 2, 0, 2),
                        Size = UDim2.new(1, -4, 1, -4),
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                        BorderColor3 = Color3.fromRGB(56, 56, 56),
                        TextColor3 = Color3.fromRGB(170, 170, 170),
                        Text = "Teleport",
                        FontFace = library.font,
                        TextSize = 12,
                        TextStrokeTransparency = 0.5,
                        TextStrokeColor3 = Color3.fromRGB(0, 0, 0),
                })
                local btn_tp = library:create("TextButton", {
                        Parent = tp_wrap,
                        Size = UDim2.new(1, 0, 1, 0),
                        BackgroundTransparency = 1,
                        Text = "",
                        ZIndex = 4,
                })
                library:connection(btn_tp.MouseEnter, function() tp_lbl.TextColor3 = themes.preset.accent end)
                library:connection(btn_tp.MouseLeave, function() tp_lbl.TextColor3 = Color3.fromRGB(170, 170, 170) end)
                btn_tp.MouseButton1Click:Connect(function()
                        local char = player.Character
                        local lchar = lp.Character
                        if char and lchar then
                                local root = char:FindFirstChild("HumanoidRootPart")
                                local lroot = lchar:FindFirstChild("HumanoidRootPart")
                                if root and lroot then
                                        lroot.CFrame = root.CFrame + Vector3.new(2, 0, 0)
                                end
                        end
                end)

                local spec_row = library:create("Frame", {
                        Parent = actions,
                        Size = UDim2.new(1, 0, 0, 16),
                        BackgroundTransparency = 1,
                })
                library:create("UIListLayout", {
                        Parent = spec_row,
                        FillDirection = Enum.FillDirection.Horizontal,
                        Padding = UDim.new(0, 4),
                })

                local spec_wrap = library:create("Frame", {
                        Parent = spec_row,
                        Size = UDim2.new(0.5, -2, 1, 0),
                        BackgroundColor3 = Color3.fromRGB(8, 8, 8),
                        BorderSizePixel = 0,
                })
                local spec_lbl = library:create("TextLabel", {
                        Parent = spec_wrap,
                        Position = UDim2.new(0, 2, 0, 2),
                        Size = UDim2.new(1, -4, 1, -4),
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                        BorderColor3 = Color3.fromRGB(56, 56, 56),
                        TextColor3 = Color3.fromRGB(170, 170, 170),
                        Text = "Spectate",
                        FontFace = library.font,
                        TextSize = 12,
                        TextStrokeTransparency = 0.5,
                        TextStrokeColor3 = Color3.fromRGB(0, 0, 0),
                })
                local btn_spec = library:create("TextButton", {
                        Parent = spec_wrap,
                        Size = UDim2.new(1, 0, 1, 0),
                        BackgroundTransparency = 1,
                        Text = "",
                        ZIndex = 4,
                })
                library:connection(btn_spec.MouseEnter, function() spec_lbl.TextColor3 = themes.preset.accent end)
                library:connection(btn_spec.MouseLeave, function() spec_lbl.TextColor3 = Color3.fromRGB(170, 170, 170) end)
                btn_spec.MouseButton1Click:Connect(function()
                        local char = player.Character
                        if char then
                                camera.CameraSubject = char:FindFirstChildOfClass("Humanoid") or char:FindFirstChild("HumanoidRootPart")
                                camera.CameraType = Enum.CameraType.Follow
                        end
                end)

                local unspec_wrap = library:create("Frame", {
                        Parent = spec_row,
                        Size = UDim2.new(0.5, -2, 1, 0),
                        BackgroundColor3 = Color3.fromRGB(8, 8, 8),
                        BorderSizePixel = 0,
                })
                local unspec_lbl = library:create("TextLabel", {
                        Parent = unspec_wrap,
                        Position = UDim2.new(0, 2, 0, 2),
                        Size = UDim2.new(1, -4, 1, -4),
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                        BorderColor3 = Color3.fromRGB(56, 56, 56),
                        TextColor3 = Color3.fromRGB(170, 170, 170),
                        Text = "Unspec",
                        FontFace = library.font,
                        TextSize = 12,
                        TextStrokeTransparency = 0.5,
                        TextStrokeColor3 = Color3.fromRGB(0, 0, 0),
                })
                local btn_unspec = library:create("TextButton", {
                        Parent = unspec_wrap,
                        Size = UDim2.new(1, 0, 1, 0),
                        BackgroundTransparency = 1,
                        Text = "",
                        ZIndex = 4,
                })
                library:connection(btn_unspec.MouseEnter, function() unspec_lbl.TextColor3 = themes.preset.accent end)
                library:connection(btn_unspec.MouseLeave, function() unspec_lbl.TextColor3 = Color3.fromRGB(170, 170, 170) end)
                btn_unspec.MouseButton1Click:Connect(function()
                        camera.CameraSubject = lp.Character and lp.Character:FindFirstChildOfClass("Humanoid")
                        camera.CameraType = Enum.CameraType.Custom
                end)

                local copy_row = library:create("Frame", {
                        Parent = actions,
                        Size = UDim2.new(1, 0, 0, 16),
                        BackgroundTransparency = 1,
                })
                library:create("UIListLayout", {
                        Parent = copy_row,
                        FillDirection = Enum.FillDirection.Horizontal,
                        Padding = UDim.new(0, 4),
                })

                local cpname_wrap = library:create("Frame", {
                        Parent = copy_row,
                        Size = UDim2.new(0.5, -2, 1, 0),
                        BackgroundColor3 = Color3.fromRGB(8, 8, 8),
                        BorderSizePixel = 0,
                })
                local cpname_lbl = library:create("TextLabel", {
                        Parent = cpname_wrap,
                        Position = UDim2.new(0, 2, 0, 2),
                        Size = UDim2.new(1, -4, 1, -4),
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                        BorderColor3 = Color3.fromRGB(56, 56, 56),
                        TextColor3 = Color3.fromRGB(170, 170, 170),
                        Text = "Copy Name",
                        FontFace = library.font,
                        TextSize = 12,
                        TextStrokeTransparency = 0.5,
                        TextStrokeColor3 = Color3.fromRGB(0, 0, 0),
                })
                local btn_cpname = library:create("TextButton", {
                        Parent = cpname_wrap,
                        Size = UDim2.new(1, 0, 1, 0),
                        BackgroundTransparency = 1,
                        Text = "",
                        ZIndex = 4,
                })
                library:connection(btn_cpname.MouseEnter, function() cpname_lbl.TextColor3 = themes.preset.accent end)
                library:connection(btn_cpname.MouseLeave, function() cpname_lbl.TextColor3 = Color3.fromRGB(170, 170, 170) end)
                btn_cpname.MouseButton1Click:Connect(function()
                        pcall(function() setclipboard(player.Name) end)
                        library:notification({ text = "Copied: " .. player.Name })
                end)

                local cpid_wrap = library:create("Frame", {
                        Parent = copy_row,
                        Size = UDim2.new(0.5, -2, 1, 0),
                        BackgroundColor3 = Color3.fromRGB(8, 8, 8),
                        BorderSizePixel = 0,
                })
                local cpid_lbl = library:create("TextLabel", {
                        Parent = cpid_wrap,
                        Position = UDim2.new(0, 2, 0, 2),
                        Size = UDim2.new(1, -4, 1, -4),
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                        BorderColor3 = Color3.fromRGB(56, 56, 56),
                        TextColor3 = Color3.fromRGB(170, 170, 170),
                        Text = "Copy ID",
                        FontFace = library.font,
                        TextSize = 12,
                        TextStrokeTransparency = 0.5,
                        TextStrokeColor3 = Color3.fromRGB(0, 0, 0),
                })
                local btn_cpid = library:create("TextButton", {
                        Parent = cpid_wrap,
                        Size = UDim2.new(1, 0, 1, 0),
                        BackgroundTransparency = 1,
                        Text = "",
                        ZIndex = 4,
                })
                library:connection(btn_cpid.MouseEnter, function() cpid_lbl.TextColor3 = themes.preset.accent end)
                library:connection(btn_cpid.MouseLeave, function() cpid_lbl.TextColor3 = Color3.fromRGB(170, 170, 170) end)
                btn_cpid.MouseButton1Click:Connect(function()
                        pcall(function() setclipboard(tostring(player.UserId)) end)
                        library:notification({ text = "Copied ID: " .. player.UserId })
                end)


                local is_expanded = false
                local function set_expanded(bool)
                        is_expanded = bool
                        expanded.Visible = bool
                        if bool then
                                TextButton.Size = UDim2.new(1, 0, 0, 124)
                                TextButton.BackgroundTransparency = 0.15
                                left_accent.Size = UDim2.new(0, 2, 0, 124)
                                left_accent.BackgroundColor3 = themes.preset.accent
                                row_stroke.Color = themes.preset.accent
                        else
                                TextButton.Size = UDim2.new(1, 0, 0, 24)
                                TextButton.BackgroundTransparency = 0.5
                                left_accent.Size = UDim2.new(0, 2, 0, 24)
                                left_accent.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
                                row_stroke.Color = Color3.fromRGB(45, 45, 45)
                        end
                end

                library:connection(TextButton.MouseEnter, function()
                        left_accent.BackgroundColor3 = themes.preset.accent
                        row_stroke.Color = themes.preset.accent
                end)

                library:connection(TextButton.MouseLeave, function()
                        if not is_expanded then
                                left_accent.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
                                row_stroke.Color = Color3.fromRGB(45, 45, 45)
                        end
                end)

                TextButton.MouseButton1Click:Connect(function()
                        if selected_button and selected_button ~= TextButton then
                                local collapse_fn = selected_button:FindFirstChild("Collapse")
                                if collapse_fn then
                                        collapse_fn:Fire()
                                end
                        end

                        selected_button = TextButton
                        selected_player = player

                        set_expanded(not is_expanded)
                end)

                local collapse_event = Instance.new("BindableEvent")
                collapse_event.Name = "Collapse"
                collapse_event.Parent = TextButton
                collapse_event.Event:Connect(function()
                        set_expanded(false)
                end)
        end

        for _, player in next, players:GetPlayers() do
                create_player(player)
        end

        library:connection(players.PlayerAdded, function(player)
                create_player(player)
        end)

        library:connection(players.PlayerRemoving, function(player)
                if player_buttons[player.Name] and player_buttons[player.Name].instance then
                        player_buttons[player.Name].instance:Destroy()
                        player_buttons[player.Name] = nil
                end
        end)

        local old_kblist = library:create("Frame", {
                Parent = library.gui,
                Name = "KeybindList",
                Active = true,
                Position = UDim2.new(0, 20, 0.5, 0),
                BackgroundColor3 = themes.preset.bg,
                BorderSizePixel = 0,
                ZIndex = 1000,
                Size = UDim2.new(0, 150, 0, 0),
                AutomaticSize = Enum.AutomaticSize.Y,
        })
        library:make_draggable(old_kblist)
        library:apply_theme(old_kblist, "bg", "BackgroundColor3")

        local kb_stroke = library:create("UIStroke", {
                Parent = old_kblist,
                Color = themes.preset.outline,
                Thickness = 1,
                LineJoinMode = Enum.LineJoinMode.Miter,
        })
        library:apply_theme(kb_stroke, "outline", "Color")

        local accent_line = library:create("Frame", {
                Parent = old_kblist,
                Name = "",
                Size = UDim2.new(1, 0, 0, 2),
                BorderSizePixel = 0,
                BackgroundColor3 = themes.preset.accent,
        })
        library:apply_theme(accent_line, "accent", "BackgroundColor3")

        local glow = library:create("ImageLabel", {
                Parent = old_kblist,
                Name = "",
                ImageColor3 = themes.preset.accent,
                ScaleType = Enum.ScaleType.Slice,
                ImageTransparency = 0.85,
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                Image = "http://www.roblox.com/asset/?id=18245826428",
                BackgroundTransparency = 1,
                Position = UDim2.new(0, -20, 0, -20),
                Size = UDim2.new(1, 40, 1, 40),
                ZIndex = 2,
                BorderSizePixel = 0,
                SliceCenter = Rect.new(Vector2.new(21, 21), Vector2.new(79, 79)),
        })
        library:apply_theme(glow, "accent", "ImageColor3")

        local name = library:create("TextLabel", {
                Parent = old_kblist,
                Name = "",
                FontFace = library.font,
                TextColor3 = Color3.fromRGB(255, 255, 255),
                Text = "keybinds",
                TextStrokeTransparency = 0,
                TextStrokeColor3 = Color3.fromRGB(0, 0, 0),
                BackgroundTransparency = 1,
                BorderSizePixel = 0,
                TextSize = 12,
                Size = UDim2.new(1, -20, 0, 16),
                Position = UDim2.new(0, 10, 0, 4),
                TextXAlignment = Enum.TextXAlignment.Left,
        })
        library:apply_theme(name, "accent", "TextColor3")

        local key_container = library:create("Frame", {
                Parent = old_kblist,
                Name = "",
                BackgroundTransparency = 1,
                BorderSizePixel = 0,
                Position = UDim2.new(0, 0, 0, 22),
                Size = UDim2.new(1, 0, 0, 0),
                AutomaticSize = Enum.AutomaticSize.Y,
        })

        library:create("UIPadding", {
                Parent = key_container,
                PaddingTop = UDim.new(0, 4),
                PaddingBottom = UDim.new(0, 6),
                PaddingLeft = UDim.new(0, 10),
                PaddingRight = UDim.new(0, 10),
        })

        library:create("UIListLayout", {
                Parent = key_container,
                Name = "",
                SortOrder = Enum.SortOrder.LayoutOrder,
                HorizontalAlignment = Enum.HorizontalAlignment.Center,
                Padding = UDim.new(0, 3),
        })

        library.keybind_path = key_container
        library._keybindListFrame = old_kblist
        library.keybind_list_enabled = true

        function cfg.toggle_list(bool)
                library.keybind_list_enabled = bool
                library:update_keybind_list_visibility()
        end

        function cfg.toggle_playerlist(bool)
                playerlist.Visible = bool
        end

        function cfg.toggle_watermark(bool)
                __holder.Visible = bool
        end

        function cfg.set_menu_visibility(bool, pl)
                WINDOW_PATH.Visible = bool

                local show_pl = flags["player_list"]
                if show_pl == nil then
                        show_pl = true
                end
                playerlist.Visible = show_pl and bool or false
        end

        return setmetatable(cfg, library)
end

function library:update_keybind_list_visibility()
        if not library._keybindListFrame then return end
        if not library.keybind_list_enabled then
                library._keybindListFrame.Visible = false
                return
        end
        local any_active = false
        for _, child in next, library.keybind_path:GetChildren() do
                if child:IsA("TextLabel") and child.Visible then
                        any_active = true
                        break
                end
        end
        library._keybindListFrame.Visible = any_active
end

function library:new_keybind(properties)
        local cfg = {
                text = properties.name or properties.text or "aimbot",
                key = properties.key or nil,
                mode = properties.mode or "hold",
        }

        local keybind_text = library:create("TextLabel", {
                Parent = library.keybind_path,
                Name = "",
                FontFace = library.font,
                LineHeight = 1.2000000476837158,
                TextStrokeTransparency = 0.5,
                AnchorPoint = Vector2.new(0.5, 0),
                TextSize = 12,
                Size = UDim2.new(1, 0, 0, 11),
                TextColor3 = Color3.fromRGB(170, 170, 170),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Text = "",
                BackgroundTransparency = 1,
                Position = UDim2.new(0.5, 0, 0, 8),
                BorderSizePixel = 0,
                Visible = true,
                TextYAlignment = Enum.TextYAlignment.Top,
                TextXAlignment = Enum.TextXAlignment.Center,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local UIPadding = library:create("UIPadding", {
                Parent = keybind_text,
                Name = "",
                PaddingTop = UDim.new(0, 6),
        })

        function cfg.set_visible(bool)
                keybind_text.Visible = bool
                library:update_keybind_list_visibility()
        end

        function cfg.change_text(text)
                keybind_text.Text = text
        end

        function keyName(key)
                local text = tostring(key) ~= "Enums" and (keys[key] or tostring(key):gsub("Enum.", "")) or nil
                local __text = text and (tostring(text):gsub("KeyCode.", ""):gsub("UserInputType.", ""))

                return __text or "..."
        end

        function cfg.update(n_properties)
                cfg.change_text(
                        "["
                                .. tostring(keyName(n_properties.key))
                                .. "] "
                                .. tostring(n_properties.text)
                                .. " ("
                                .. tostring(n_properties.mode)
                                .. ")"
                )
        end

        cfg.change_text(
                "[" .. tostring(keyName(cfg.key)) .. "] " .. tostring(cfg.text) .. " (" .. tostring(cfg.mode) .. ")"
        )

        return cfg
end

function library:notification(properties)
        local cfg = {
                time = properties.time or 5,
                text = properties.text or properties.name or "ledger.live is pasted",
        }


        function cfg:refresh_notifications()
                local p = library.notification_position or "Top Left"
                local isBot = p:find("Bottom") ~= nil
                local isCtr = p:find("Center") ~= nil
                local isRt  = p:find("Right")  ~= nil
                local rxs = isRt and 1 or (isCtr and 0.5 or 0)
                local rxo = isRt and -20 or 20
                local rys = isBot and 1 or 0
                for idx, notif in next, library.notifications do
                        local ryo = isBot and (-72 - (idx-1)*28) or (72 + (idx-1)*28)
                        tween_service
                                :Create(
                                        notif,
                                        TweenInfo.new(0.3, Enum.EasingStyle.Exponential, Enum.EasingDirection.InOut),
                                        { Position = dim2(rxs, rxo, rys, ryo) }
                                )
                                :Play()
                end
        end

        local _p   = library.notification_position or "Top Left"
        local _isBot = _p:find("Bottom") ~= nil
        local _isCtr = _p:find("Center") ~= nil
        local _isRt  = _p:find("Right")  ~= nil
        local _xScale = _isRt and 1 or (_isCtr and 0.5 or 0)
        local _xOff   = _isRt and -20 or 20
        local _yScale = _isBot and 1 or 0
        local _yOff   = _isBot and (-72 - #library.notifications * 28) or (72 + #library.notifications * 28)
        local _anchorOff = _isCtr and Vector2.new(0.5, 0) or (_isRt and Vector2.new(0, 0) or Vector2.new(1, 0))
        local _anchorOn  = _isCtr and Vector2.new(0.5, 0) or (_isRt and Vector2.new(1, 0) or Vector2.new(0, 0))

        local holder = library:create("Frame", {
                Parent = library.gui,
                Name = "",
                BackgroundTransparency = 1,
                Position = UDim2.new(_xScale, _xOff, _yScale, _yOff),
                BorderColor3 = Color3.fromRGB(19, 19, 19),
                ZIndex = 2,
                AutomaticSize = Enum.AutomaticSize.X,
                BackgroundColor3 = themes.preset.contrast,
                AnchorPoint = _anchorOff,
        })

        local inline1 = library:create("Frame", {
                Parent = holder,
                Name = "",
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Size = UDim2.new(0, 0, 0, 24),
                AutomaticSize = Enum.AutomaticSize.X,
                BackgroundColor3 = themes.preset.contrast,
        })

        local inline2 = library:create("Frame", {
                Parent = inline1,
                Name = "",
                Position = UDim2.new(0, 0, 0, 2),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Size = UDim2.new(1, -4, 1, -4),
                BorderSizePixel = 0,
                BackgroundColor3 = themes.preset.bg,
        })

        local main = library:create("Frame", {
                Parent = inline2,
                Name = "",
                Position = UDim2.new(0, 2, 0, 2),
                BorderColor3 = themes.preset.outline,
                Size = UDim2.new(1, -4, 1, -4),
                BackgroundColor3 = themes.preset.bg,
        })

        local tab_inline = library:create("Frame", {
                Parent = main,
                Name = "",
                Position = UDim2.new(0, 2, 0, 2),
                BorderColor3 = themes.preset.outline,
                Size = UDim2.new(1, -4, 1, -4),
                BorderSizePixel = 0,
                BackgroundColor3 = themes.preset.inline,
        })

        local name = library:create("TextLabel", {
                Parent = tab_inline,
                Name = "",
                FontFace = library.font,
                TextColor3 = themes.preset.text,
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Text = cfg.text,
                TextStrokeTransparency = 0.5,
                Size = UDim2.new(0, 0, 1, 0),
                Position = UDim2.new(0, 8, 0, 0),
                BackgroundTransparency = 1,
                TextXAlignment = Enum.TextXAlignment.Left,
                BorderSizePixel = 0,
                AutomaticSize = Enum.AutomaticSize.X,
                TextSize = 12,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local UIPadding = library:create("UIPadding", {
                Parent = tab_inline,
                Name = "",
                PaddingRight = UDim.new(0, 14),
        })

        local depth = library:create("Frame", {
                Parent = inline1,
                Name = "",
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                BackgroundTransparency = 0.5,
                Position = UDim2.new(0, 1, 0, 0),
                Size = UDim2.new(0, 1, 1, 0),
                ZIndex = 2,
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(0, 0, 0),
        })

        local accent_line = library:create("Frame", {
                Parent = inline1,
                Name = "",
                BorderColor3 = Color3.fromRGB(34, 34, 34),
                Size = UDim2.new(0, 2, 1, 0),
                BorderSizePixel = 0,
                BackgroundColor3 = themes.preset.accent,
        })

        library:apply_theme(accent_line, "accent", "BackgroundColor3")

        local glow = library:create("ImageLabel", {
                Parent = holder,
                Name = "",
                ImageColor3 = themes.preset.accent,
                ScaleType = Enum.ScaleType.Slice,
                ImageTransparency = 0.8999999761581421,
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                Image = "http://www.roblox.com/asset/?id=18245826428",
                BackgroundTransparency = 1,
                Position = UDim2.new(0, -20, 0, 0),
                Size = UDim2.new(0, 42, 1, 40),
                ZIndex = 2,
                BorderSizePixel = 0,
                SliceCenter = Rect.new(Vector2.new(21, 21), Vector2.new(79, 79)),
        })

        library:apply_theme(glow, "accent", "ImageColor3")

        -- Text-only mode: strip all decoration
        if library.notification_text_only then
                for _, f in next, { inline1, inline2, main, tab_inline, depth } do
                        f.BackgroundTransparency = 1
                        f.BorderSizePixel = 0
                end
                accent_line.Visible = false
                glow.Visible = false
        end

        local _anim = _isCtr and "Fade" or (library.notification_animation or "Slide")

        task.spawn(function()
                if _anim == "Slide" then
                        tween_service:Create(holder, TweenInfo.new(1, Enum.EasingStyle.Quad, Enum.EasingDirection.InOut), { AnchorPoint = _anchorOn }):Play()
                elseif _anim == "Fade" then
                        holder.AnchorPoint = _anchorOn
                        for _, v in next, holder:GetDescendants() do
                                if v:IsA("TextLabel") then v.TextTransparency = 1
                                elseif v:IsA("Frame") and v.BackgroundTransparency < 1 then v.BackgroundTransparency = 1
                                elseif v:IsA("ImageLabel") then v.ImageTransparency = 1
                                end
                        end
                        tween_service:Create(name, TweenInfo.new(0.5, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), { TextTransparency = 0 }):Play()
                        if not library.notification_text_only then
                                tween_service:Create(inline1, TweenInfo.new(0.5), { BackgroundTransparency = 0 }):Play()
                                tween_service:Create(inline2, TweenInfo.new(0.5), { BackgroundTransparency = 0 }):Play()
                                tween_service:Create(main,   TweenInfo.new(0.5), { BackgroundTransparency = 0 }):Play()
                                tween_service:Create(tab_inline, TweenInfo.new(0.5), { BackgroundTransparency = 0 }):Play()
                        end
                else
                        holder.AnchorPoint = _anchorOn
                end

                task.wait(cfg.time)

                if _anim == "Slide" then
                        tween_service:Create(holder, TweenInfo.new(1, Enum.EasingStyle.Quad, Enum.EasingDirection.InOut), { AnchorPoint = _anchorOff }):Play()
                elseif _anim == "Fade" then
                        for _, v in next, holder:GetDescendants() do
                                if v:IsA("TextLabel") then
                                        tween_service:Create(v, TweenInfo.new(0.5), { TextTransparency = 1 }):Play()
                                elseif v:IsA("Frame") and v.BackgroundTransparency < 1 then
                                        tween_service:Create(v, TweenInfo.new(0.5), { BackgroundTransparency = 1 }):Play()
                                elseif v:IsA("ImageLabel") then
                                        tween_service:Create(v, TweenInfo.new(0.5), { ImageTransparency = 1 }):Play()
                                end
                        end
                else
                        holder.AnchorPoint = _anchorOff
                end
        end)

        task.delay(cfg.time + 0.1, function()
                if library and library.notifications then
                        local idx = table.find(library.notifications, holder)
                        if idx then table.remove(library.notifications, idx) end
                end
                if cfg.refresh_notifications then pcall(function() cfg:refresh_notifications() end) end
                task.wait(0.5)
                pcall(function() holder:Destroy() end)
        end)

        if library and library.notifications then
                table.insert(library.notifications, holder)
        end
end

function library:tab(properties)
        local cfg = {
                name = properties.name or "tab",
                enabled = false,
        }

        local TAB_BUTTON = library:create("TextButton", {
                Parent = self.tab_holder,
                Name = "",
                FontFace = library.font,
                TextColor3 = themes.preset.unselected_text,
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Text = cfg.name,
                TextStrokeTransparency = 0.5,
                BackgroundTransparency = 1,
                Size = UDim2.new(0.3330000042915344, -4, 0, 22),
                BorderSizePixel = 0,
                TextSize = 12,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })
        library:apply_theme(TAB_BUTTON, "unselected_text", "TextColor3")
        library:apply_theme(TAB_BUTTON, "text", "TextColor3")

        local line = library:create("Frame", {
                Parent = TAB_BUTTON,
                Name = "",
                Position = UDim2.new(0, 0, 1, 0),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Size = UDim2.new(1, 0, 0, 2),
                BorderSizePixel = 0,
                BackgroundColor3 = rgb(57, 57, 57),
        })

        library:apply_theme(line, "accent", "BackgroundColor3")

        local glow = library:create("ImageLabel", {
                Parent = line,
                Name = "",
                ImageColor3 = themes.preset.accent,
                ScaleType = Enum.ScaleType.Slice,
                ImageTransparency = 0.8999999761581421,
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                Image = "http://www.roblox.com/asset/?id=18245826428",
                BackgroundTransparency = 1,
                Position = UDim2.new(0, -20, 0, -20),
                Size = UDim2.new(1, 40, 1, 40),
                ZIndex = 2,
                Visible = false,
                BorderSizePixel = 0,
                SliceCenter = Rect.new(Vector2.new(21, 21), Vector2.new(79, 79)),
        })

        library:apply_theme(glow, "accent", "ImageColor3")

        local depth = library:create("Frame", {
                Parent = line,
                Name = "",
                BackgroundTransparency = 0.5,
                Position = UDim2.new(0, 0, 0, 1),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Size = UDim2.new(1, 0, 0, 1),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(0, 0, 0),
        })

        local TAB = library:create("Frame", {
                Parent = self.tab_instance_holder,
                Name = "",
                Visible = false,
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                BackgroundTransparency = 1,
                Size = UDim2.new(1, 0, 1, 0),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local scrolling_columns = library:create("Frame", {
                Parent = TAB,
                Name = "",
                ClipsDescendants = true,
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 6, 0, 6),
                Size = UDim2.new(1, -12, 1, -12),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        cfg["column_holder"] = scrolling_columns

        local UIListLayout = library:create("UIListLayout", {
                Parent = scrolling_columns,
                Name = "",
                FillDirection = Enum.FillDirection.Horizontal,
                HorizontalFlex = Enum.UIFlexAlignment.Fill,
                Padding = UDim.new(0, 5),
                SortOrder = Enum.SortOrder.LayoutOrder,
        })

        local left = library:create("ScrollingFrame", {
                Parent = scrolling_columns,
                Name = "",
                ScrollBarImageColor3 = Color3.fromRGB(0, 0, 0),
                Active = true,
                AutomaticCanvasSize = Enum.AutomaticSize.Y,
                ScrollBarThickness = 0,
                Size = UDim2.new(0.5, -3, 1, 0),
                ClipsDescendants = true,
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 4, 0, 0),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                BorderSizePixel = 0,
                CanvasSize = UDim2.new(0, 0, 0, 0),
        })

        left:GetPropertyChangedSignal("CanvasPosition"):Connect(function()
                if library.current_element_open then
                        library.current_element_open.set_visible(false)
                        library.current_element_open.open = false
                        library.current_element_open = nil
                end
        end)

        local right1 = library:create("ScrollingFrame", {
                Parent = scrolling_columns,
                Name = "",
                ScrollBarImageColor3 = Color3.fromRGB(0, 0, 0),
                Active = true,
                AutomaticCanvasSize = Enum.AutomaticSize.Y,
                ScrollBarThickness = 0,
                Size = UDim2.new(0.5, -3, 1, 0),
                ClipsDescendants = true,
                BackgroundTransparency = 1,
                Position = UDim2.new(0.5, 0, 0, 0),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                BorderSizePixel = 0,
                CanvasSize = UDim2.new(0, 0, 0, 0),
        })

        right1:GetPropertyChangedSignal("CanvasPosition"):Connect(function()
                if library.current_element_open then
                        library.current_element_open.set_visible(false)
                        library.current_element_open.open = false
                        library.current_element_open = nil
                end
        end)

        local function _updateCols1()
                local hasRight = false
                for _, c in ipairs(right1:GetChildren()) do
                        if not c:IsA("UIListLayout") and not c:IsA("UIPadding") then hasRight = true; break end
                end
                if hasRight then
                        left.Size = UDim2.new(0.5, -3, 1, 0)
                        right1.Visible = true
                else
                        left.Size = UDim2.new(1, 0, 1, 0)
                        right1.Visible = false
                end
        end
        right1.ChildAdded:Connect(_updateCols1)
        right1.ChildRemoved:Connect(_updateCols1)
        task.defer(_updateCols1)

        cfg["left"] = left
        cfg["right"] = right1

        local UIListLayout = library:create("UIListLayout", {
                Parent = left,
                Name = "",
                Padding = UDim.new(0, 6),
                SortOrder = Enum.SortOrder.LayoutOrder,
        })

        local UIPadding = library:create("UIPadding", {
                Parent = left,
                Name = "",
                PaddingBottom = UDim.new(0, 15),
        })

        library:create("UIListLayout", {
                Parent = right1,
                Name = "",
                Padding = UDim.new(0, 6),
                SortOrder = Enum.SortOrder.LayoutOrder,
        })

        library:create("UIPadding", {
                Parent = right1,
                Name = "",
                PaddingBottom = UDim.new(0, 15),
        })

        function cfg.open_tab()
                if library.current_tab and library.current_tab[1] ~= TAB_BUTTON then
                        local button = library.current_tab[1]
                        button.TextColor3 = themes.preset.unselected_text

                        local parent = button:FindFirstChildOfClass("Frame")
                        parent.BackgroundColor3 = rgb(57, 57, 57)
                        parent:FindFirstChildOfClass("ImageLabel").Visible = false

                        library.current_tab[2].Visible = false
                end

                library.current_tab = {
                        TAB_BUTTON,
                        TAB,
                }

                line.BackgroundColor3 = themes.preset.accent
                glow.Visible = true
                TAB_BUTTON.TextColor3 = themes.preset.text
                TAB.Visible = true

                if library.current_element_open and library.current_element_open ~= cfg then
                        library.current_element_open.set_visible(false)
                        library.current_element_open.open = false
                        library.current_element_open = nil
                end
        end

        TAB_BUTTON.MouseButton1Click:Connect(cfg.open_tab)

        return setmetatable(cfg, library)
end

function library:subtab(properties)
        local cfg = {
                name = properties.name or "subtab",
                parent_tab = properties.parent_tab or self,
        }

        local tabObj = cfg.parent_tab
        local tabFrame = tabObj.column_holder.Parent
        local colHolder = tabObj.column_holder

        local wrapper = tabFrame:FindFirstChild("_subtab_wrapper")
        if not wrapper then
                colHolder.Visible = false

                wrapper = library:create("Frame", {
                        Parent = tabFrame,
                        Name = "_subtab_wrapper",
                        Position = colHolder.Position,
                        Size = colHolder.Size,
                        BackgroundTransparency = 1,
                        BorderSizePixel = 0,
                        ZIndex = 2,
                })

                local bar = library:create("Frame", {
                        Parent = wrapper,
                        Name = "_subtab_bar",
                        Position = UDim2.new(0, 0, 0, 0),
                        Size = UDim2.new(1, 0, 0, 22),
                        BackgroundTransparency = 1,
                        BorderSizePixel = 0,
                })

                library:create("UIListLayout", {
                        Parent = bar,
                        Name = "",
                        FillDirection = Enum.FillDirection.Horizontal,
                        HorizontalAlignment = Enum.HorizontalAlignment.Left,
                        Padding = UDim.new(0, 4),
                        SortOrder = Enum.SortOrder.LayoutOrder,
                })
        end

        local bar = wrapper:FindFirstChild("_subtab_bar")

        local btn = library:create("TextButton", {
                Parent = bar,
                Name = "",
                FontFace = library.font,
                TextColor3 = themes.preset.unselected_text,
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Text = cfg.name,
                TextStrokeTransparency = 0.5,
                BackgroundTransparency = 1,
                Size = UDim2.new(0, 80, 1, 0),
                BorderSizePixel = 0,
                TextSize = 11,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local contentArea = wrapper:FindFirstChild("_subtab_content_area") or library:create("Frame", {
                Parent = wrapper,
                Name = "_subtab_content_area",
                Position = UDim2.new(0, 0, 0, 26),
                Size = UDim2.new(1, 0, 1, -26),
                BackgroundTransparency = 1,
                BorderSizePixel = 0,
        })

        local content = library:create("Frame", {
                Parent = contentArea,
                Name = "",
                BackgroundTransparency = 1,
                BorderSizePixel = 0,
                Size = UDim2.new(1, 0, 1, 0),
                Visible = false,
        })

        local left = library:create("ScrollingFrame", {
                Parent = content,
                Name = "",
                ScrollBarImageColor3 = Color3.fromRGB(0, 0, 0),
                Active = true,
                AutomaticCanvasSize = Enum.AutomaticSize.Y,
                ScrollBarThickness = 0,
                Size = UDim2.new(0.5, -6, 1, 0),
                ClipsDescendants = true,
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 4, 0, 0),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                BorderSizePixel = 0,
                CanvasSize = UDim2.new(0, 0, 0, 0),
        })

        left:GetPropertyChangedSignal("CanvasPosition"):Connect(function()
                if library.current_element_open then
                        library.current_element_open.set_visible(false)
                        library.current_element_open.open = false
                        library.current_element_open = nil
                end
        end)

        library:create("UIListLayout", {
                Parent = left,
                Name = "",
                Padding = UDim.new(0, 6),
                SortOrder = Enum.SortOrder.LayoutOrder,
        })

        library:create("UIPadding", {
                Parent = left,
                Name = "",
                PaddingBottom = UDim.new(0, 15),
        })

        local right = library:create("ScrollingFrame", {
                Parent = content,
                Name = "",
                ScrollBarImageColor3 = Color3.fromRGB(0, 0, 0),
                Active = true,
                AutomaticCanvasSize = Enum.AutomaticSize.Y,
                ScrollBarThickness = 0,
                Size = UDim2.new(0.5, -6, 1, 0),
                ClipsDescendants = true,
                BackgroundTransparency = 1,
                Position = UDim2.new(0.5, 2, 0, 0),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                BorderSizePixel = 0,
                CanvasSize = UDim2.new(0, 0, 0, 0),
        })

        right:GetPropertyChangedSignal("CanvasPosition"):Connect(function()
                if library.current_element_open then
                        library.current_element_open.set_visible(false)
                        library.current_element_open.open = false
                        library.current_element_open = nil
                end
        end)

        library:create("UIListLayout", {
                Parent = right,
                Name = "",
                Padding = UDim.new(0, 6),
                SortOrder = Enum.SortOrder.LayoutOrder,
        })

        library:create("UIPadding", {
                Parent = right,
                Name = "",
                PaddingBottom = UDim.new(0, 15),
        })

        local function _updateCols2()
                local hasRight = false
                for _, c in ipairs(right:GetChildren()) do
                        if not c:IsA("UIListLayout") and not c:IsA("UIPadding") then hasRight = true; break end
                end
                if hasRight then
                        left.Size = UDim2.new(0.5, -6, 1, 0)
                        right.Visible = true
                else
                        left.Size = UDim2.new(1, -8, 1, 0)
                        right.Visible = false
                end
        end
        right.ChildAdded:Connect(_updateCols2)
        right.ChildRemoved:Connect(_updateCols2)
        task.defer(_updateCols2)

        cfg.left = left
        cfg.right = right
        cfg.button = btn
        cfg.content = content

        function cfg.open()
                for _, child in ipairs(contentArea:GetChildren()) do
                        if child ~= content and child:IsA("Frame") then
                                child.Visible = false
                        end
                end
                content.Visible = true
                for _, child in ipairs(bar:GetChildren()) do
                        if child:IsA("TextButton") then
                                child.TextColor3 = themes.preset.unselected_text
                        end
                end
                btn.TextColor3 = themes.preset.text
        end

        btn.MouseButton1Click:Connect(cfg.open)

        local isFirst = true
        for _, child in ipairs(bar:GetChildren()) do
                if child:IsA("TextButton") and child ~= btn then
                        isFirst = false
                        break
                end
        end
        if isFirst then
                cfg.open()
        end

        return setmetatable(cfg, library)
end

function library:section(properties)
        local cfg = {
                name = properties.name or properties.Name or "Section",
                side = properties.side or properties.Side or "left",
        }

        local section = library:create("Frame", {
                Parent = self[cfg.side],
                Name = "",
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                BackgroundTransparency = 1,
                BorderSizePixel = 0,
                Size = UDim2.new(1, 0, 0, 0),
                ZIndex = 2,
                AutomaticSize = Enum.AutomaticSize.Y,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local section_inline = library:create("Frame", {
                Parent = section,
                Name = "",
                Position = UDim2.new(0, 0, 0, 4),
                BorderColor3 = Color3.fromRGB(19, 19, 19),
                Size = UDim2.new(1, 0, 1, -4),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(8, 8, 8),
        })
        library:apply_theme(section_inline, "contrast", "BackgroundColor3")

        local name = library:create("TextLabel", {
                Parent = section_inline,
                Name = "",
                FontFace = library.font,
                TextColor3 = Color3.fromRGB(170, 170, 170),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Text = cfg.name,
                TextStrokeTransparency = 0.5,
                BorderSizePixel = 0,
                Size = UDim2.new(1, 0, 0, 1),
                BackgroundTransparency = 1,
                TextXAlignment = Enum.TextXAlignment.Left,
                Position = UDim2.new(0, 8, 0, 0),
                ZIndex = 2,
                TextSize = 12,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })
        library:apply_theme(name, "text", "TextColor3")

        local section = library:create("Frame", {
                Parent = section_inline,
                Name = "",
                Position = UDim2.new(0, 2, 0, 2),
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Size = UDim2.new(1, -4, 1, -4),
                BackgroundColor3 = themes.preset.bg,
        })
        library:apply_theme(section, "bg", "BackgroundColor3")
        library:apply_theme(section, "outline", "BorderColor3")

        local elements = library:create("Frame", {
                Parent = section,
                Name = "",
                Position = UDim2.new(0, 12, 0, 12),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Size = UDim2.new(1, -24, 0, 0),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local UIListLayout = library:create("UIListLayout", {
                Parent = elements,
                Name = "",
                SortOrder = Enum.SortOrder.LayoutOrder,
                HorizontalAlignment = Enum.HorizontalAlignment.Center,
                Padding = UDim.new(0, 3),
        })

        local UIPadding = library:create("UIPadding", {
                Parent = section,
                Name = "",
                PaddingBottom = UDim.new(0, 13),
        })

        cfg["holder"] = elements

        return setmetatable(cfg, library)
end

function library:hitpart_picker(properties)
        local cfg = {
                name = properties.name or properties.Name or "Hitpart",
                side = properties.side or properties.Side or "left",
                flag = properties.flag or "Hitpart",
                default = properties.default or { "Head" },
                type_char = properties.type or "R6",
                multi = properties.multi or false,
                callback = properties.callback or function() end,
                previous_holder = self,
        }

        flags[cfg.flag] = {}

        local bodyparts = {}
        local bools = {}

        local r15_hitpart_holder = library:create("Frame", {
                Parent = self[cfg.side],
                Name = "",
                BackgroundTransparency = 1,
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Size = UDim2.new(1, 0, 0, 272),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local hitpart_inline = library:create("Frame", {
                Parent = r15_hitpart_holder,
                Name = "",
                Position = UDim2.new(0, 0, 0, 4),
                BorderColor3 = Color3.fromRGB(19, 19, 19),
                Size = UDim2.new(1, 0, 1, -4),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(8, 8, 8),
        })

        local hitpart = library:create("Frame", {
                Parent = hitpart_inline,
                Name = "",
                Position = UDim2.new(0, 2, 0, 2),
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Size = UDim2.new(1, -4, 1, -4),
                BackgroundColor3 = themes.preset.bg,
        })
        library:apply_theme(hitpart, "bg", "BackgroundColor3")

        if cfg.type_char == "R15" then
                bodyparts.Head = library:create("TextButton", {
                        Text = "",
                        Parent = hitpart,
                        Name = "",
                        Position = UDim2.new(0.5, -25, 0, 16),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(0, 50, 0, 44),
                        BorderSizePixel = 0,
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                })

                bodyparts.UpperTorso = library:create("TextButton", {
                        Text = "",
                        Parent = hitpart,
                        Name = "",
                        Position = UDim2.new(0.5, -42, 0, 64),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(0, 84, 0, 76),
                        BorderSizePixel = 0,
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                })

                bodyparts.LeftUpperArm = library:create("TextButton", {
                        Text = "",
                        Parent = hitpart,
                        Name = "",
                        Position = UDim2.new(0.5, -86, 0, 64),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(0, 40, 0, 34),
                        BorderSizePixel = 0,
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                })

                bodyparts.RightUpperArm = library:create("TextButton", {
                        Text = "",
                        Parent = hitpart,
                        Name = "",
                        Position = UDim2.new(0.5, 46, 0, 64),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(0, 40, 0, 34),
                        BorderSizePixel = 0,
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                })

                bodyparts.LeftUpperLeg = library:create("TextButton", {
                        Text = "",
                        Parent = hitpart,
                        Name = "",
                        Position = UDim2.new(0.5, -42, 0, 158),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(0, 40, 0, 34),
                        BorderSizePixel = 0,
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                })

                bodyparts.LeftLowerLeg = library:create("TextButton", {
                        Text = "",
                        Parent = hitpart,
                        Name = "",
                        Position = UDim2.new(0.5, -42, 0, 196),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(0, 40, 0, 42),
                        BorderSizePixel = 0,
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                })

                bodyparts.RightFoot = library:create("TextButton", {
                        Text = "",
                        Parent = hitpart,
                        Name = "",
                        Position = UDim2.new(0.5, 2, 0, 242),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(0, 40, 0, 6),
                        BorderSizePixel = 0,
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                })

                bodyparts.LeftFoot = library:create("TextButton", {
                        Text = "",
                        Parent = hitpart,
                        Name = "",
                        Position = UDim2.new(0.5, -42, 0, 242),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(0, 40, 0, 6),
                        BorderSizePixel = 0,
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                })

                bodyparts.RightLowerLeg = library:create("TextButton", {
                        Text = "",
                        Parent = hitpart,
                        Name = "",
                        Position = UDim2.new(0.5, 2, 0, 196),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(0, 40, 0, 42),
                        BorderSizePixel = 0,
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                })

                bodyparts.RightUpperLeg = library:create("TextButton", {
                        Text = "",
                        Parent = hitpart,
                        Name = "",
                        Position = UDim2.new(0.5, 2, 0, 158),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(0, 40, 0, 34),
                        BorderSizePixel = 0,
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                })

                bodyparts.LeftHand = library:create("TextButton", {
                        Text = "",
                        Parent = hitpart,
                        Name = "",
                        Position = UDim2.new(0.5, -86, 0, 148),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(0, 40, 0, 6),
                        BorderSizePixel = 0,
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                })

                bodyparts.RightHand = library:create("TextButton", {
                        Text = "",
                        Parent = hitpart,
                        Name = "",
                        Position = UDim2.new(0.5, 46, 0, 148),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(0, 40, 0, 6),
                        BorderSizePixel = 0,
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                })

                bodyparts.LowerTorso = library:create("TextButton", {
                        Text = "",
                        Parent = hitpart,
                        Name = "",
                        Position = UDim2.new(0.5, -42, 0, 144),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(0, 84, 0, 10),
                        BorderSizePixel = 0,
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                })

                bodyparts.RightLowerArm = library:create("TextButton", {
                        Text = "",
                        Parent = hitpart,
                        Name = "",
                        Position = UDim2.new(0.5, 46, 0, 102),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(0, 40, 0, 42),
                        BorderSizePixel = 0,
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                })

                bodyparts.LeftLowerArm = library:create("TextButton", {
                        Text = "",
                        Parent = hitpart,
                        Name = "",
                        Position = UDim2.new(0.5, -86, 0, 102),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(0, 40, 0, 42),
                        BorderSizePixel = 0,
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                })

                local outline = library:create("TextButton", {
                        Text = "",
                        Parent = hitpart,
                        Name = "",
                        Position = UDim2.new(0.5, -10, 0, 96),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(0, 20, 0, 20),
                        BorderSizePixel = 0,
                        BackgroundColor3 = Color3.fromRGB(22, 22, 22),
                })

                bodyparts.HumanoidRootPart = library:create("TextButton", {
                        Text = "",
                        Parent = outline,
                        Name = "",
                        Position = UDim2.new(0, 4, 0, 4),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(1, -8, 1, -8),
                        BorderSizePixel = 0,
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                })
        else
                bodyparts.Head = library:create("TextButton", {
                        Parent = hitpart,
                        Name = "",
                        Text = "",
                        Position = UDim2.new(0.5, -25, 0, 16),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(0, 50, 0, 44),
                        BorderSizePixel = 0,
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                })

                bodyparts.Torso = library:create("TextButton", {
                        Parent = hitpart,
                        Name = "",
                        Text = "",
                        Position = UDim2.new(0.5, -42, 0, 64),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(0, 84, 0, 90),
                        BorderSizePixel = 0,
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                })

                bodyparts.LeftArm = library:create("TextButton", {
                        Parent = hitpart,
                        Name = "",
                        Position = UDim2.new(0.5, -86, 0, 64),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(0, 40, 0, 90),
                        BorderSizePixel = 0,
                        Text = "",
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                })

                bodyparts.RightArm = library:create("TextButton", {
                        Parent = hitpart,
                        Name = "",
                        Position = UDim2.new(0.5, 46, 0, 64),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(0, 40, 0, 90),
                        Text = "",
                        BorderSizePixel = 0,
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                })

                bodyparts.RightLeg = library:create("TextButton", {
                        Parent = hitpart,
                        Name = "",
                        Position = UDim2.new(0.5, 2, 0, 158),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(0, 40, 0, 90),
                        BorderSizePixel = 0,
                        Text = "",
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                })

                bodyparts.LeftLeg = library:create("TextButton", {
                        Parent = hitpart,
                        Name = "",
                        Position = UDim2.new(0.5, -42, 0, 158),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(0, 40, 0, 90),
                        BorderSizePixel = 0,
                        Text = "",
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                })

                local hrp_out = library:create("TextButton", {
                        Parent = hitpart,
                        Name = "",
                        Position = UDim2.new(0.5, -10, 0, 99),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(0, 20, 0, 20),
                        BorderSizePixel = 0,
                        Text = "",
                        BackgroundColor3 = Color3.fromRGB(22, 22, 22),
                })

                bodyparts.HumanoidRootPart = library:create("TextButton", {
                        Parent = hrp_out,
                        Name = "",
                        Position = UDim2.new(0, 4, 0, 4),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(1, -8, 1, -8),
                        BorderSizePixel = 0,
                        Text = "",
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                })
        end

        local name = library:create("TextLabel", {
                Parent = hitpart_inline,
                Name = "",
                FontFace = library.font,
                TextColor3 = Color3.fromRGB(170, 170, 170),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Text = cfg.name,
                TextStrokeTransparency = 0.5,
                BorderSizePixel = 0,
                Size = UDim2.new(1, 0, 0, 1),
                BackgroundTransparency = 1,
                TextXAlignment = Enum.TextXAlignment.Left,
                Position = UDim2.new(0, 8, 0, 0),
                ZIndex = 2,
                TextSize = 12,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        function cfg.set(parts)
                flags[cfg.flag] = {}
                for name, button in pairs(bodyparts) do
                        bools[name] = false
                        local glow = button:FindFirstChildOfClass("ImageLabel")
                        if glow then
                                glow.Visible = false
                        end
                        button.BackgroundColor3 = Color3.fromRGB(38, 38, 38)
                end

                for _, part in pairs(parts) do
                        if bodyparts[part] then
                                bools[part] = true
                                table.insert(flags[cfg.flag], part)
                                local glow = bodyparts[part]:FindFirstChildOfClass("ImageLabel")
                                if glow then
                                        glow.Visible = true
                                end
                                bodyparts[part].BackgroundColor3 = themes.preset.accent
                        end
                end

                cfg.callback(flags[cfg.flag])
        end

        for name, button in next, bodyparts do
                bools[name] = false

                library:apply_theme(button, "accent", "BackgroundColor3")

                local glow = library:create("ImageLabel", {
                        Parent = button,
                        Name = "",
                        Visible = false,
                        ImageColor3 = themes.preset.accent,
                        ScaleType = Enum.ScaleType.Slice,
                        ImageTransparency = 0.8999999761581421,
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                        Image = "http://www.roblox.com/asset/?id=18245826428",
                        BackgroundTransparency = 1,
                        Position = UDim2.new(0, -20, 0, -20),
                        Size = UDim2.new(1, 40, 1, 40),
                        ZIndex = 2,
                        BorderSizePixel = 0,
                        SliceCenter = Rect.new(Vector2.new(21, 21), Vector2.new(79, 79)),
                })

                library:apply_theme(glow, "accent", "ImageColor3")

                library:connection(button.MouseButton1Click, function()
                        if not cfg.multi then
                                cfg.set({ name })
                        else
                                bools[name] = not bools[name]

                                if bools[name] then
                                        table.insert(flags[cfg.flag], name)
                                else
                                        local index = table.find(flags[cfg.flag], name)
                                        table.remove(flags[cfg.flag], index)
                                end

                                glow.Visible = bools[name]
                                button.BackgroundColor3 = bools[name] and themes.preset.accent or Color3.fromRGB(38, 38, 38)

                                cfg.callback(flags[cfg.flag])
                        end
                end)
        end

        if #cfg.default > 1 and not cfg.multi then
                cfg.default = { cfg.default[1] }
        end

        cfg.set(cfg.default)
        config_flags[cfg.flag] = cfg.set
        return setmetatable(cfg, library)
end

function library:toggle(properties)
        local cfg = {
                enabled = properties.enabled or nil,
                name = properties.name or "Toggle",
                flag = properties.flag or tostring(math.random(1, 9999999)),
                callback = properties.callback or function() end,
                default = properties.default or false,
                previous_holder = self,
        }

        local object = library:create("TextButton", {
                Parent = self.holder,
                Name = "",
                FontFace = library.font,
                TextColor3 = Color3.fromRGB(170, 170, 170),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Text = cfg.name,
                TextStrokeTransparency = 0.5,
                BorderSizePixel = 0,
                BackgroundTransparency = 1,
                TextXAlignment = Enum.TextXAlignment.Left,
                Size = UDim2.new(1, -26, 0, 12),
                ZIndex = 1,
                TextSize = 12,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })
        library:apply_theme(object, "text", "TextColor3")

        local right_components = library:create("Frame", {
                Parent = object,
                Name = "",
                Position = UDim2.new(1, 15, 0, 1),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Size = UDim2.new(0, 0, 1, 0),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local list = library:create("UIListLayout", {
                Parent = right_components,
                Name = "",
                FillDirection = Enum.FillDirection.Horizontal,
                HorizontalAlignment = Enum.HorizontalAlignment.Right,
                Padding = UDim.new(0, 3),
                SortOrder = Enum.SortOrder.LayoutOrder,
        })

        local icon_inline = library:create("TextButton", {
                Parent = object,
                Name = "",
                Position = UDim2.new(0, -15, 0, 1),
                BorderColor3 = Color3.fromRGB(19, 19, 19),
                Size = UDim2.new(0, 10, 0, 10),
                BorderSizePixel = 0,
                Text = "",
                AutoButtonColor = false,
                BackgroundColor3 = Color3.fromRGB(8, 8, 8),
        })
        library:apply_theme(icon_inline, "contrast", "BackgroundColor3")

        local icon = library:create("Frame", {
                Parent = icon_inline,
                Name = "",
                Position = UDim2.new(0, 2, 0, 2),
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Size = UDim2.new(1, -4, 1, -4),
                BackgroundColor3 = themes.preset.bg,
        })
        library:apply_theme(icon, "bg", "BackgroundColor3")

        local icon_2 = library:create("Frame", {
                Parent = icon,
                Name = "",
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Size = UDim2.new(1, 0, 1, 0),
                BackgroundColor3 = themes.preset.accent,
        })
        library:apply_theme(icon_2, "accent", "BackgroundColor3")

        local glow = library:create("ImageLabel", {
                Parent = icon_inline,
                Name = "",
                Visible = false,
                ImageColor3 = themes.preset.accent,
                ScaleType = Enum.ScaleType.Slice,
                ImageTransparency = 0.75,
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                Image = "http://www.roblox.com/asset/?id=18245826428",
                BackgroundTransparency = 1,
                Position = UDim2.new(0, -12, 0, -12),
                Size = UDim2.new(1, 24, 1, 24),
                ZIndex = 2,
                BorderSizePixel = 0,
                SliceCenter = Rect.new(Vector2.new(21, 21), Vector2.new(79, 79)),
        })

        library:apply_theme(glow, "accent", "ImageColor3")

        local bottom_components = library:create("Frame", {
                Parent = object,
                Name = "",
                Visible = true,
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Position = UDim2.new(0, 0, 0, 13),
                Size = UDim2.new(1, 26, 0, 0),
                ZIndex = 2,
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local list = library:create("UIListLayout", {
                Parent = bottom_components,
                Name = "",
                Padding = UDim.new(0, 4),
                SortOrder = Enum.SortOrder.LayoutOrder,
        })

        function cfg.set(bool)
                cfg.enabled = bool
                icon_2.Visible = bool
                glow.Visible = bool

                flags[cfg.flag] = bool

                cfg.callback(bool)
        end

        function cfg.set_visible(bool)
                object.Visible = bool
        end

        library:connection(object.MouseButton1Click, function()
                cfg.enabled = not cfg.enabled

                cfg.set(cfg.enabled)
        end)

        library:connection(icon_inline.MouseButton1Click, function()
                cfg.enabled = not cfg.enabled

                cfg.set(cfg.enabled)
        end)

        cfg.set(cfg.default)

        cfg.object = object

        self.previous_holder = left_components
        self.bottom_holder = bottom_components
        self.right_holder = right_components

        config_flags[cfg.flag] = cfg.set

        return setmetatable(cfg, library)
end

function library:slider(properties)
        local cfg = {
                name = properties.name or nil,
                suffix = properties.suffix or "",
                flag = properties.flag or tostring(2 ^ 789),
                callback = properties.callback or function() end,

                min = properties.min or properties.minimum or 0,
                max = properties.max or properties.maximum or 100,
                intervals = properties.interval or properties.decimal or 1,
                default = properties.default or 10,

                dragging = false,
                value = properties.default or 10,

                previous_holder = self,
        }

        local object
        local bottom_components
        if cfg.name then
                object = library:create("TextLabel", {
                        Parent = self.holder,
                        Name = "",
                        FontFace = library.font,
                        TextColor3 = Color3.fromRGB(170, 170, 170),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Text = cfg.name,
                        TextStrokeTransparency = 0.5,
                        Size = UDim2.new(1, -26, 0, 12),
                        BorderSizePixel = 0,
                        BackgroundTransparency = 1,
                        TextXAlignment = Enum.TextXAlignment.Left,
                        AutomaticSize = Enum.AutomaticSize.Y,
                        TextYAlignment = Enum.TextYAlignment.Top,
                        TextSize = 12,
                        BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                })

                bottom_components = library:create("Frame", {
                        Parent = object,
                        Name = "",
                        Visible = true,
                        Position = UDim2.new(0, 0, 0, 13),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(1, 26, 0, 0),
                        BorderSizePixel = 0,
                        BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                })

                local list = library:create("UIListLayout", {
                        Parent = bottom_components,
                        Name = "",
                        Padding = UDim.new(0, 4),
                        SortOrder = Enum.SortOrder.LayoutOrder,
                })
        else
                self.bottom_holder.Parent.AutomaticSize = Enum.AutomaticSize.Y
                self.bottom_holder.Parent.TextYAlignment = Enum.TextYAlignment.Top
        end

        local slider_holder = library:create("Frame", {
                Parent = cfg.name and bottom_components or self.bottom_holder,
                Name = "",
                BackgroundTransparency = 1,
                Size = UDim2.new(1, 0, 0, 0),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                BorderSizePixel = 0,
                AutomaticSize = Enum.AutomaticSize.Y,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local slider_inline = library:create("TextButton", {
                Parent = slider_holder,
                Name = "",
                Position = UDim2.new(0, 0, 0, 1),
                BorderColor3 = Color3.fromRGB(19, 19, 19),
                Size = UDim2.new(1, -26, 0, 8),
                BorderSizePixel = 0,
                Text = "",
                AutoButtonColor = false,
                BackgroundColor3 = Color3.fromRGB(8, 8, 8),
        })
        library:apply_theme(slider_inline, "contrast", "BackgroundColor3")

        local fill_inline = library:create("Frame", {
                Parent = slider_inline,
                Name = "",
                Size = UDim2.new(0.5, 0, 1, 0),
                BorderColor3 = Color3.fromRGB(19, 19, 19),
                ZIndex = 2,
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(19, 19, 19),
        })
        library:apply_theme(fill_inline, "inline", "BackgroundColor3")

        local fill = library:create("Frame", {
                Parent = fill_inline,
                Name = "",
                Position = UDim2.new(0, 2, 0, 2),
                Size = UDim2.new(1, 0, 1, -4),
                BackgroundColor3 = themes.preset.accent,
        })

        library:apply_theme(fill, "accent", "BackgroundColor3")
        library:apply_theme(fill, "accent", "BorderColor3")

        local VALUE_TEXT = library:create("TextLabel", {
                Parent = fill_inline,
                Name = "",
                RichText = true,
                TextColor3 = Color3.fromRGB(170, 170, 170),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                TextStrokeTransparency = 0.5,
                Size = UDim2.new(0, 1, 0, 11),
                BackgroundTransparency = 1,
                Position = UDim2.new(1, 0, 0, 1),
                BorderSizePixel = 0,
                FontFace = library.font,
                TextSize = 12,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })
        library:apply_theme(VALUE_TEXT, "text", "TextColor3")

        local glow = library:create("ImageLabel", {
                Parent = fill_inline,
                Name = "",
                ImageColor3 = themes.preset.accent,
                ScaleType = Enum.ScaleType.Slice,
                ImageTransparency = 0.8999999761581421,
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                Image = "http://www.roblox.com/asset/?id=18245826428",
                BackgroundTransparency = 1,
                Position = UDim2.new(0, -18, 0, -18),
                Size = UDim2.new(1, 36, 1, 36),
                ZIndex = 2,
                BorderSizePixel = 0,
                SliceCenter = Rect.new(Vector2.new(21, 21), Vector2.new(79, 79)),
        })

        library:apply_theme(glow, "accent", "ImageColor3")

        local add = library:create("TextButton", {
                Parent = slider_inline,
                Name = "",
                TextWrapped = true,
                TextColor3 = Color3.fromRGB(170, 170, 170),
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Text = "+",
                TextStrokeTransparency = 0.5,
                BackgroundTransparency = 1,
                Position = UDim2.new(1, 5, 0, -1),
                Size = UDim2.new(0, 8, 0, 8),
                FontFace = library.font,
                TextSize = 8,
                BackgroundColor3 = Color3.fromRGB(38, 38, 38),
        })

        local sub = library:create("TextButton", {
                Parent = slider_inline,
                Name = "",
                TextWrapped = true,
                TextColor3 = Color3.fromRGB(170, 170, 170),
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Text = "-",
                TextStrokeTransparency = 0.5,
                BackgroundTransparency = 1,
                Position = UDim2.new(0, -15, 0, -1),
                Size = UDim2.new(0, 8, 0, 8),
                FontFace = library.font,
                TextSize = 12,
                BackgroundColor3 = Color3.fromRGB(38, 38, 38),
        })

        local slider = library:create("Frame", {
                Parent = slider_inline,
                Name = "",
                Position = UDim2.new(0, 2, 0, 2),
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Size = UDim2.new(1, -4, 1, -4),
                BackgroundColor3 = themes.preset.bg,
        })
        library:apply_theme(slider, "bg", "BackgroundColor3")

        local pad = library:create("UIPadding", {
                Parent = slider_holder,
                Name = "",
                PaddingBottom = UDim.new(0, -17),
        })

        function cfg.set(value)
                if type(value) ~= "number" then
                        return
                end

                cfg.value = math.clamp(library:round(value, cfg.intervals), cfg.min, cfg.max)

                fill_inline.Size = dim2((cfg.value - cfg.min) / (cfg.max - cfg.min), 0, 1, 0)
                VALUE_TEXT.Text = tostring(cfg.value) .. cfg.suffix
                flags[cfg.flag] = cfg.value

                cfg.callback(flags[cfg.flag])
        end

        function cfg.set_visible(bool)
                if object then object.Visible = bool end
        end

        library:connection(uis.InputChanged, function(input)
                if cfg.dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
                        local size_x = (input.Position.X - slider.AbsolutePosition.X) / slider.AbsoluteSize.X
                        local value = ((cfg.max - cfg.min) * size_x) + cfg.min
                        cfg.set(value)
                end
        end)

        library:connection(uis.InputEnded, function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 then
                        cfg.dragging = false
                end
        end)

        slider_inline.MouseButton1Down:Connect(function()
                cfg.dragging = true
        end)

        add.MouseButton1Down:Connect(function()
                cfg.value += cfg.intervals
                cfg.set(cfg.value)
        end)

        sub.MouseButton1Down:Connect(function()
                cfg.value -= cfg.intervals
                cfg.set(cfg.value)
        end)

        cfg.set(cfg.default)

        cfg.object = object

        config_flags[cfg.flag] = cfg.set

        library.config_flags[cfg.flag] = cfg.set

        return setmetatable(cfg, library)
end

function library:dropdown(properties)
        local cfg = {
                name = properties.name or nil,
                flag = properties.flag or tostring(math.random(1, 9999999)),

                items = properties.items or { "1", "2", "3" },
                callback = properties.callback or function() end,
                multi = properties.multi or false,

                open = false,
                option_instances = {},
                multi_items = {},

                previous_holder = self,
        }
        cfg.default = properties.default or (cfg.multi and { cfg.items[1] }) or cfg.items[1] or nil

        local bottom_components
        local object
        if cfg.name then
                object = library:create("TextLabel", {
                        Parent = self.holder,
                        Name = "",
                        FontFace = library.font,
                        TextColor3 = Color3.fromRGB(170, 170, 170),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Text = cfg.name,
                        TextStrokeTransparency = 0.5,
                        Size = UDim2.new(1, -26, 0, 12),
                        BorderSizePixel = 0,
                        ZIndex = 2,
                        BackgroundTransparency = 1,
                        TextXAlignment = Enum.TextXAlignment.Left,
                        AutomaticSize = Enum.AutomaticSize.Y,
                        TextYAlignment = Enum.TextYAlignment.Top,
                        TextSize = 12,
                        BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                })

                bottom_components = library:create("Frame", {
                        Parent = object,
                        Name = "",
                        Visible = true,
                        Position = UDim2.new(0, 0, 0, 13),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(1, 26, 0, 0),
                        BorderSizePixel = 0,
                        BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                })

                local list = library:create("UIListLayout", {
                        Parent = bottom_components,
                        Name = "",
                        Padding = UDim.new(0, 4),
                        SortOrder = Enum.SortOrder.LayoutOrder,
                })
        else
                self.bottom_holder.Parent.AutomaticSize = Enum.AutomaticSize.Y
                self.bottom_holder.Parent.TextYAlignment = Enum.TextYAlignment.Top
        end

        local dropdown_inline = library:create("Frame", {
                Parent = cfg.name and bottom_components or self.bottom_holder,
                Name = "",
                Position = UDim2.new(0, -15, 0, 2),
                BorderColor3 = Color3.fromRGB(19, 19, 19),
                Size = UDim2.new(1, -26, 0, 16),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(8, 8, 8),
        })
        library:apply_theme(dropdown_inline, "contrast", "BackgroundColor3")

        local dropdown = library:create("TextButton", {
                Parent = dropdown_inline,
                Name = "",
                FontFace = library.font,
                TextColor3 = Color3.fromRGB(170, 170, 170),
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Text = "option 1, option 3",
                TextStrokeTransparency = 0.5,
                TextXAlignment = Enum.TextXAlignment.Left,
                Size = UDim2.new(1, -4, 1, -4),
                Position = UDim2.new(0, 2, 0, 2),
                TextSize = 12,
                BackgroundColor3 = Color3.fromRGB(38, 38, 38),
        })
        library:apply_theme(dropdown, "text", "TextColor3")
        library:apply_theme(dropdown, "contrast", "BackgroundColor3")

        local UIPadding = library:create("UIPadding", {
                Parent = dropdown,
                Name = "",
                PaddingLeft = UDim.new(0, 5),
        })

        local icon = library:create("TextLabel", {
                Parent = dropdown,
                Name = "",
                FontFace = library.font,
                TextColor3 = Color3.fromRGB(170, 170, 170),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Text = "+",
                TextStrokeTransparency = 0.5,
                Size = UDim2.new(0, 1, 1, 0),
                BackgroundTransparency = 1,
                TextXAlignment = Enum.TextXAlignment.Right,
                Position = UDim2.new(1, -6, 0, -1),
                BorderSizePixel = 0,
                TextSize = 8,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local content_inline = library:create("Frame", {
                Parent = library.gui,
                Name = "",
                BorderColor3 = Color3.fromRGB(19, 19, 19),
                Size = UDim2.new(0, dropdown_inline.AbsoluteSize.X, 0, 0),
                Position = UDim2.new(
                        0,
                        dropdown_inline.AbsolutePosition.X,
                        0,
                        dropdown_inline.AbsolutePosition.Y + dropdown_inline.AbsoluteSize.Y + 2
                ),
                BorderSizePixel = 0,
                ZIndex = 2,
                Visible = false,
                AutomaticSize = Enum.AutomaticSize.Y,
                BackgroundColor3 = Color3.fromRGB(8, 8, 8),
        })
        library:apply_theme(content_inline, "contrast", "BackgroundColor3")

        dropdown_inline:GetPropertyChangedSignal("AbsolutePosition"):Connect(function()
                content_inline.Position = UDim2.new(
                        0,
                        dropdown_inline.AbsolutePosition.X,
                        0,
                        dropdown_inline.AbsolutePosition.Y + dropdown_inline.AbsoluteSize.Y + 2
                )
        end)

        dropdown_inline:GetPropertyChangedSignal("AbsoluteSize"):Connect(function()
                content_inline.Size = UDim2.new(0, dropdown_inline.AbsoluteSize.X, 0, 0)
        end)

        local content = library:create("Frame", {
                Parent = content_inline,
                Name = "",
                Position = UDim2.new(0, 2, 0, 2),
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Size = UDim2.new(1, -4, 1, -4),
                BackgroundColor3 = Color3.fromRGB(38, 38, 38),
        })
        library:apply_theme(content, "contrast", "BackgroundColor3")

        local options = library:create("Frame", {
                Parent = content,
                Name = "",
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 2, 0, 2),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Size = UDim2.new(1, -4, 1, -4),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(50, 50, 50),
        })

        local UIListLayout = library:create("UIListLayout", {
                Parent = options,
                Name = "",
                Padding = UDim.new(0, 2),
                SortOrder = Enum.SortOrder.LayoutOrder,
        })

        local UIPadding = library:create("UIPadding", {
                Parent = options,
                Name = "",
                PaddingBottom = UDim.new(0, 4),
        })



        function cfg.set_visible(bool)
                content_inline.Visible = bool

                icon.Text = bool and "-" or "+"
                icon.TextSize = bool and 12 or 8

                if cfg.name then
                        object.ZIndex = bool and 9999 or 3
                end

                if bool then
                        if library.current_element_open and library.current_element_open ~= cfg then
                                library.current_element_open.set_visible(false)
                                library.current_element_open.open = false
                        end

                        library.current_element_open = cfg
                end
        end

        function cfg.set(value)
                local selected = {}

                local is_table = type(value) == "table"

                for _, v in next, cfg.option_instances do
                        if v.Text == value or (is_table and table.find(value, v.Text)) then
                                table.insert(selected, v.Text)
                                cfg.multi_items = selected
                                v.BackgroundTransparency = 0
                        else
                                v.BackgroundTransparency = 1
                        end
                end

                dropdown.Text = is_table and table.concat(selected, ",  ") or selected[1] or ""
                flags[cfg.flag] = is_table and selected or selected[1]
                cfg.callback(flags[cfg.flag])
        end

        function cfg:refresh_options(refreshed_list)
                for _, v in next, cfg.option_instances do
                        v:Destroy()
                end

                cfg.option_instances = {}

                for i, v in next, refreshed_list do
                        local op3 = library:create("TextButton", {
                                Parent = options,
                                Name = "",
                                FontFace = library.font,
                                TextColor3 = Color3.fromRGB(170, 170, 170),
                                BorderColor3 = Color3.fromRGB(56, 56, 56),
                                Text = v,
                                BackgroundTransparency = 1,
                                TextStrokeTransparency = 0.5,
                                Size = UDim2.new(1, 0, 0, 14),
                                TextXAlignment = Enum.TextXAlignment.Left,
                                Position = UDim2.new(0, 2, 0, 2),
                                BorderSizePixel = 0,
                                TextSize = 12,
                                BackgroundColor3 = Color3.fromRGB(65, 65, 65),
                        })
                        library:apply_theme(op3, "text", "TextColor3")
                        library:apply_theme(op3, "inline", "BackgroundColor3")

                        local UIPadding = library:create("UIPadding", {
                                Parent = op3,
                                Name = "",
                                PaddingLeft = UDim.new(0, 5),
                        })

                        table.insert(cfg.option_instances, op3)

                        op3.MouseButton1Down:Connect(function()
                                if cfg.multi then
                                        local selected_index = table.find(cfg.multi_items, op3.Text)

                                        if selected_index then
                                                table.remove(cfg.multi_items, selected_index)
                                        else
                                                table.insert(cfg.multi_items, op3.Text)
                                        end

                                        cfg.set(cfg.multi_items)
                                else
                                        cfg.set_visible(false)
                                        cfg.open = false

                                        cfg.set(op3.Text)
                                end
                        end)
                end

                dropdown.Text = ""
        end

        dropdown.MouseButton1Click:Connect(function()
                cfg.open = not cfg.open

                cfg.set_visible(cfg.open)
        end)

        cfg:refresh_options(cfg.items)

        cfg.set(cfg.default)

        cfg.object = object

        library.config_flags[cfg.flag] = cfg.set

        return setmetatable(cfg, library)
end

function library:colorpicker(properties)
        local cfg = {
                name = properties.name or nil,
                flag = properties.flag or tostring(2 ^ 789),
                color = properties.color or properties.default or Color3.new(1, 1, 1),
                alpha = properties.alpha or 1,
                callback = properties.callback or function() end,
                animation = "normal",
                saved_color,
                right_holder = self.right_holder or nil,
                holder = self.holder or nil,
        }

        flags[cfg.flag] = {}

        local dragging_sat = false
        local dragging_hue = false
        local dragging_alpha = false

        local h, s, v = cfg.color:ToHSV()
        local a = cfg.alpha

        local object
        local right_components
        if cfg.name then
                object = library:create("TextLabel", {
                        Parent = self.holder,
                        Name = "",
                        FontFace = library.font,
                        TextColor3 = Color3.fromRGB(170, 170, 170),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Text = cfg.name,
                        TextStrokeTransparency = 0.5,
                        Size = UDim2.new(1, -26, 0, 12),
                        BorderSizePixel = 0,
                        BackgroundTransparency = 1,
                        TextXAlignment = Enum.TextXAlignment.Left,
                        TextYAlignment = Enum.TextYAlignment.Top,
                        TextSize = 12,
                        BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                })

                right_components = library:create("Frame", {
                        Parent = object,
                        Name = "",
                        Position = UDim2.new(1, 15, 0, 1),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(0, 0, 1, 0),
                        BorderSizePixel = 0,
                        BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                })

                local list = library:create("UIListLayout", {
                        Parent = right_components,
                        Name = "",
                        FillDirection = Enum.FillDirection.Horizontal,
                        HorizontalAlignment = Enum.HorizontalAlignment.Right,
                        Padding = UDim.new(0, 3),
                        SortOrder = Enum.SortOrder.LayoutOrder,
                })
        end

        local icon_inline = library:create("TextButton", {
                Parent = cfg.name and right_components or self.right_holder,
                Name = "",
                Text = "",
                Size = UDim2.new(0, 16, 0, 10),
                Position = UDim2.new(0, -15, 0, 1),
                BorderColor3 = Color3.fromRGB(19, 19, 19),
                ZIndex = 3,
                BorderSizePixel = 0,
                BackgroundColor3 = themes.preset.contrast,
        })

        local icon = library:create("Frame", {
                Parent = icon_inline,
                Name = "",
                Position = UDim2.new(0, 2, 0, 2),
                BorderColor3 = themes.preset.outline,
                ZIndex = 2,
                Size = UDim2.new(1, -4, 1, -4),
                BackgroundColor3 = cfg.color,
        })

        local glow = library:create("ImageLabel", {
                Parent = icon_inline,
                Name = "",
                ImageColor3 = themes.preset.accent,
                ScaleType = Enum.ScaleType.Slice,
                ImageTransparency = 0.8999999761581421,
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                Image = "http://www.roblox.com/asset/?id=18245826428",
                BackgroundTransparency = 1,
                Position = UDim2.new(0, -20, 0, -20),
                Size = UDim2.new(1, 40, 1, 40),
                ZIndex = 0,
                BorderSizePixel = 0,
                SliceCenter = Rect.new(Vector2.new(21, 21), Vector2.new(79, 79)),
        })
        library:apply_theme(glow, "accent", "ImageColor3")

        local picker_inline = library:create("Frame", {
                Parent = library.gui,
                Name = "",
                Size = UDim2.new(0, 142, 0, 146),
                Position = dim2(0, 0, 0, 0),
                BorderColor3 = Color3.fromRGB(19, 19, 19),
                ZIndex = 9999,
                BorderSizePixel = 0,
                Visible = false,
                BackgroundColor3 = Color3.fromRGB(8, 8, 8),
        })

        local picker = library:create("Frame", {
                Parent = picker_inline,
                Name = "",
                Position = UDim2.new(0, 2, 0, 2),
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Size = UDim2.new(1, -4, 1, -4),
                BackgroundColor3 = Color3.fromRGB(38, 38, 38),
        })

        local sat_inline = library:create("TextButton", {
                Parent = picker,
                Name = "",
                Text = "",
                Position = UDim2.new(0, 4, 0, 4),
                BorderColor3 = Color3.fromRGB(19, 19, 19),
                Size = UDim2.new(1, -8, 1, -50),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(8, 8, 8),
        })

        local sat = library:create("Frame", {
                Parent = sat_inline,
                Name = "",
                Position = UDim2.new(0, 2, 0, 2),
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Size = UDim2.new(1, -4, 1, -4),
                BackgroundColor3 = Color3.fromRGB(255, 0, 0),
        })

        local sat_white = library:create("Frame", {
                Parent = sat,
                Name = "",
                Size = UDim2.new(1, 0, 1, 0),
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                ZIndex = 2,
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local UIGradient = library:create("UIGradient", {
                Parent = sat_white,
                Name = "",
                Transparency = NumberSequence.new({
                        NumberSequenceKeypoint.new(0, 0),
                        NumberSequenceKeypoint.new(1, 1),
                }),
        })

        local sat_black = library:create("Frame", {
                Parent = sat_white,
                Name = "",
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Size = UDim2.new(1, 0, 1, 0),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local UIGradient = library:create("UIGradient", {
                Parent = sat_black,
                Name = "",
                Rotation = 90,
                Transparency = NumberSequence.new({
                        NumberSequenceKeypoint.new(0, 1),
                        NumberSequenceKeypoint.new(1, 0),
                }),
                Color = ColorSequence.new({
                        ColorSequenceKeypoint.new(0, Color3.fromRGB(0, 0, 0)),
                        ColorSequenceKeypoint.new(1, Color3.fromRGB(0, 0, 0)),
                }),
        })

        local sat_black_cursor = library:create("Frame", {
                Parent = sat_black,
                Name = "",
                Position = UDim2.new(0.800000011920929, 0, 0.20000000298023224, 0),
                BorderColor3 = Color3.fromRGB(108, 22, 22),
                Size = UDim2.new(0, 1, 0, 1),
                BackgroundColor3 = Color3.fromRGB(204, 41, 41),
        })

        local preview_inline = library:create("Frame", {
                Parent = picker,
                Name = "",
                Position = UDim2.new(1, -20, 1, -20),
                BorderColor3 = Color3.fromRGB(19, 19, 19),
                Size = UDim2.new(0, 16, 0, 16),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(35, 35, 35),
        })

        local preview = library:create("Frame", {
                Parent = preview_inline,
                Name = "",
                BackgroundTransparency = 0,
                Position = UDim2.new(0, 2, 0, 2),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                ZIndex = 2,
                Size = UDim2.new(1, -4, 1, -4),
                BackgroundColor3 = Color3.fromRGB(204, 41, 41),
        })

        local preview_image = library:create("ImageLabel", {
                Parent = preview_inline,
                Name = "",
                ScaleType = Enum.ScaleType.Tile,
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Image = "http://www.roblox.com/asset/?id=18274452449",
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 2, 0, 2),
                Size = UDim2.new(1, -4, 1, -4),
                TileSize = UDim2.new(0, 6, 0, 6),
                BorderSizePixel = 0,
                ZIndex = 3,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local hue_inline = library:create("TextButton", {
                Parent = picker,
                Text = "",
                Name = "",
                Position = UDim2.new(0, 4, 1, -44),
                BorderColor3 = Color3.fromRGB(19, 19, 19),
                Size = UDim2.new(1, -8, 0, 10),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(8, 8, 8),
        })

        local hue_border = library:create("Frame", {
                Parent = hue_inline,
                Name = "",
                Position = UDim2.new(0, 2, 0, 2),
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Size = UDim2.new(1, -4, 1, -4),
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local hue = library:create("Frame", {
                Parent = hue_border,
                Name = "",
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Size = UDim2.new(1, 0, 1, 0),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local UIGradient = library:create("UIGradient", {
                Parent = hue,
                Name = "",
                Color = ColorSequence.new({
                        ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 0, 0)),
                        ColorSequenceKeypoint.new(0.16699999570846558, Color3.fromRGB(255, 255, 0)),
                        ColorSequenceKeypoint.new(0.3330000042915344, Color3.fromRGB(0, 255, 0)),
                        ColorSequenceKeypoint.new(0.5, Color3.fromRGB(0, 255, 255)),
                        ColorSequenceKeypoint.new(0.6669999957084656, Color3.fromRGB(0, 0, 255)),
                        ColorSequenceKeypoint.new(0.8330000042915344, Color3.fromRGB(255, 0, 255)),
                        ColorSequenceKeypoint.new(1, Color3.fromRGB(255, 0, 0)),
                }),
        })

        local hue_cursor = library:create("Frame", {
                Parent = hue,
                Name = "",
                BorderColor3 = Color3.fromRGB(108, 22, 22),
                Size = UDim2.new(0, 1, 1, 0),
                BackgroundColor3 = Color3.fromRGB(204, 41, 41),
        })

        local input_inline = library:create("Frame", {
                Parent = picker,
                Name = "",
                Position = UDim2.new(0, 4, 1, -20),
                BorderColor3 = Color3.fromRGB(19, 19, 19),
                Size = UDim2.new(1, -26, 0, 16),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(8, 8, 8),
        })

        local __input = library:create("TextBox", {
                Parent = input_inline,
                Name = "",
                FontFace = library.font,
                TextColor3 = Color3.fromRGB(170, 170, 170),
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Text = "204, 41, 41, 0.5",
                TextStrokeTransparency = 0.5,
                Size = UDim2.new(1, -4, 1, -4),
                PlaceholderColor3 = Color3.fromRGB(90, 90, 90),
                Position = UDim2.new(0, 2, 0, 2),
                PlaceholderText = "r, g, b, a",
                TextSize = 12,
                BackgroundColor3 = Color3.fromRGB(38, 38, 38),
        })

        local alpha_inline = library:create("TextButton", {
                Parent = picker,
                Name = "",
                Text = "",
                Position = UDim2.new(0, 4, 1, -32),
                BorderColor3 = Color3.fromRGB(19, 19, 19),
                Size = UDim2.new(1, -8, 0, 10),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(8, 8, 8),
        })

        local alpha = library:create("Frame", {
                Parent = alpha_inline,
                Name = "",
                Position = UDim2.new(0, 2, 0, 2),
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Size = UDim2.new(1, -4, 1, -4),
                BackgroundColor3 = Color3.fromRGB(204, 41, 41),
        })

        local alpha_image = library:create("ImageLabel", {
                Parent = alpha,
                Name = "",
                ScaleType = Enum.ScaleType.Tile,
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Image = "http://www.roblox.com/asset/?id=18343135386",
                BackgroundTransparency = 1,
                Size = UDim2.new(1, 0, 1, 0),
                TileSize = UDim2.new(0, 6, 0, 6),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local UIGradient = library:create("UIGradient", {
                Parent = alpha_image,
                Name = "",
                Transparency = NumberSequence.new({
                        NumberSequenceKeypoint.new(0, 1),
                        NumberSequenceKeypoint.new(1, 0),
                }),
        })

        local alpha_cursor = library:create("Frame", {
                Parent = alpha_image,
                Name = "",
                Position = UDim2.new(0.5, 0, 0, 0),
                BorderColor3 = Color3.fromRGB(108, 22, 22),
                Size = UDim2.new(0, 1, 1, 0),
                BackgroundColor3 = Color3.fromRGB(204, 41, 41),
        })


        local content_inline = library:create("Frame", {
                Parent = library.gui,
                Name = "",
                BorderColor3 = Color3.fromRGB(19, 19, 19),
                Size = UDim2.new(0, 73, 0, 0),
                Position = dim2(0, icon_inline.AbsolutePosition.X + 20, 0, icon_inline.AbsolutePosition.Y),
                BorderSizePixel = 0,
                ZIndex = 2,
                Visible = false,
                AutomaticSize = Enum.AutomaticSize.Y,
                BackgroundColor3 = Color3.fromRGB(8, 8, 8),
        })

        local content = library:create("Frame", {
                Parent = content_inline,
                Name = "",
                Position = UDim2.new(0, 2, 0, 2),
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Size = UDim2.new(1, -4, 1, -4),
                BackgroundColor3 = Color3.fromRGB(38, 38, 38),
        })

        local options = library:create("Frame", {
                Parent = content,
                Name = "",
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 2, 0, 2),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Size = UDim2.new(1, -4, 1, -4),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(50, 50, 50),
        })

        local UIListLayout = library:create("UIListLayout", {
                Parent = options,
                Name = "",
                Padding = UDim.new(0, 2),
                SortOrder = Enum.SortOrder.LayoutOrder,
        })

        local normal = library:create("TextButton", {
                Parent = options,
                Name = "",
                FontFace = library.font,
                TextColor3 = Color3.fromRGB(170, 170, 170),
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Text = "normal",
                TextStrokeTransparency = 0.5,
                Size = UDim2.new(1, 0, 0, 12),
                TextXAlignment = Enum.TextXAlignment.Left,
                Position = UDim2.new(0, 2, 0, 2),
                BorderSizePixel = 0,
                TextSize = 12,
                BackgroundColor3 = Color3.fromRGB(65, 65, 65),
        })

        local UIPadding = library:create("UIPadding", {
                Parent = normal,
                Name = "",
                PaddingBottom = UDim.new(0, 1),
                PaddingLeft = UDim.new(0, 5),
        })

        local rainbow = library:create("TextButton", {
                Parent = options,
                Name = "",
                FontFace = library.font,
                TextColor3 = Color3.fromRGB(170, 170, 170),
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Text = "rainbow",
                TextStrokeTransparency = 0.5,
                Size = UDim2.new(1, 0, 0, 12),
                BackgroundTransparency = 1,
                TextXAlignment = Enum.TextXAlignment.Left,
                Position = UDim2.new(0, 2, 0, 2),
                BorderSizePixel = 0,
                TextSize = 12,
                BackgroundColor3 = Color3.fromRGB(65, 65, 65),
        })

        local UIPadding = library:create("UIPadding", {
                Parent = rainbow,
                Name = "",
                PaddingBottom = UDim.new(0, 1),
                PaddingLeft = UDim.new(0, 5),
        })

        local fade = library:create("TextButton", {
                Parent = options,
                Name = "",
                FontFace = library.font,
                TextColor3 = Color3.fromRGB(170, 170, 170),
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Text = "fade",
                TextStrokeTransparency = 0.5,
                Size = UDim2.new(1, 0, 0, 12),
                BackgroundTransparency = 1,
                TextXAlignment = Enum.TextXAlignment.Left,
                Position = UDim2.new(0, 2, 0, 2),
                BorderSizePixel = 0,
                TextSize = 12,
                BackgroundColor3 = Color3.fromRGB(65, 65, 65),
        })

        local UIPadding = library:create("UIPadding", {
                Parent = fade,
                Name = "",
                PaddingBottom = UDim.new(0, 1),
                PaddingLeft = UDim.new(0, 5),
        })

        local UIPadding = library:create("UIPadding", {
                Parent = options,
                Name = "",
                PaddingBottom = UDim.new(0, 4),
        })

        local fade_alpha = library:create("TextButton", {
                Parent = options,
                Name = "",
                FontFace = library.font,
                TextColor3 = Color3.fromRGB(170, 170, 170),
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Text = "fade alpha",
                TextStrokeTransparency = 0.5,
                Size = UDim2.new(1, 0, 0, 12),
                BackgroundTransparency = 1,
                TextXAlignment = Enum.TextXAlignment.Left,
                Position = UDim2.new(0, 2, 0, 2),
                BorderSizePixel = 0,
                TextSize = 12,
                BackgroundColor3 = Color3.fromRGB(65, 65, 65),
        })

        local UIPadding = library:create("UIPadding", {
                Parent = fade_alpha,
                Name = "",
                PaddingBottom = UDim.new(0, 1),
                PaddingLeft = UDim.new(0, 5),
        })

        function cfg.set_visible(bool)
                local px = icon_inline.AbsolutePosition.X + 1
                local py = icon_inline.AbsolutePosition.Y + 17
                local pickerH = 146
                local viewH = 600
                pcall(function()
                        local cam = workspace.CurrentCamera
                        if cam then viewH = cam.ViewportSize.Y end
                end)
                if py + pickerH > viewH then
                        py = icon_inline.AbsolutePosition.Y - pickerH - 2
                end
                if py < 0 then py = 2 end
                picker_inline.Position = dim2(0, px, 0, py)
                content_inline.Position = dim2(0, icon_inline.AbsolutePosition.X + 20, 0, icon_inline.AbsolutePosition.Y)

                picker_inline.Visible = bool
                content_inline.Visible = false

                if bool then
                        if library.current_element_open and library.current_element_open ~= cfg then
                                library.current_element_open.set_visible(false)
                                library.current_element_open.open = false
                        end

                        library.current_element_open = cfg
                end
        end

        icon_inline.MouseButton1Click:Connect(function()
                cfg.open = not cfg.open

                cfg.set_visible(cfg.open)
        end)

        icon_inline.MouseButton2Click:Connect(function()
                if cfg.open then
                        cfg.open = false
                        cfg.set_visible(false)
                end

                content_inline.Visible = not content_inline.Visible

                local px = icon_inline.AbsolutePosition.X + 1
                local py = icon_inline.AbsolutePosition.Y + 17
                local pickerH = 146
                local viewH = 600
                pcall(function()
                        local cam = workspace.CurrentCamera
                        if cam then viewH = cam.ViewportSize.Y end
                end)
                if py + pickerH > viewH then
                        py = icon_inline.AbsolutePosition.Y - pickerH - 2
                end
                if py < 0 then py = 2 end
                picker_inline.Position = dim2(0, px, 0, py)
                content_inline.Position = dim2(0, icon_inline.AbsolutePosition.X + 20, 0, icon_inline.AbsolutePosition.Y)
        end)

        function cfg.set(color, alpha)
                if color then
                        h, s, v = color:ToHSV()
                else
                        cfg.saved_color = hsv(h, s, v)
                end

                if alpha then
                        a = alpha
                end

                local visual = alpha_inline:FindFirstChildOfClass("Frame")

                if not visual then
                        return
                end

                local hsv_position = Color3.fromHSV(h, s, v)
                local Color = Color3.fromHSV(h, s, v)

                local value = h
                hue_cursor.Position = dim2(value, 0, 0, 0)

                alpha_cursor.Position = dim2(a, 0, 0, 0)

                visual.BackgroundColor3 = Color


                local RGB_Format = visual.BackgroundColor3

                icon_inline.BackgroundColor3 = Color3.fromRGB(RGB_Format.R / 4, RGB_Format.G / 4, RGB_Format.B / 4)
                icon.BorderColor3 = Color3.fromRGB(
                        math.floor((Color.R * 255) + 0.5) / 2,
                        math.floor((Color.G * 255) + 0.5) / 2,
                        math.floor((Color.B * 255) + 0.5) / 2
                )
                icon.BackgroundColor3 = Color

                __input.Text = math.floor(RGB_Format.R * 255)
                        .. ", "
                        .. math.floor(RGB_Format.G * 255)
                        .. ", "
                        .. math.floor(RGB_Format.B * 255)
                        .. ", "
                        .. library:round(a, 0.01)
                preview.BackgroundColor3 = Color
                preview_image.ImageTransparency = 1 - a

                sat.BackgroundColor3 = Color3.fromHSV(h, 1, 1)

                sat_black_cursor.Position = dim2(s, 0, 1 - v, 0)

                cfg.color = Color
                cfg.alpha = a

                flags[cfg.flag] = {
                        Color = Color,
                        Transparency = a,
                }
                cfg.saved_color = hsv(h, s, v)

                cfg.callback(Color, a)
        end

        __input.FocusLost:Connect(function()
                local text = __input.Text
                local r, g, b, a = library:convert_string_rgb(text)

                if r and g and b and a then
                        cfg.set(rgb(r, g, b), a)
                end
        end)

        function cfg.update_color()
                local mouse = uis:GetMouseLocation()

                if dragging_sat then
                        s = math.clamp(
                                (vec2(mouse.X, mouse.Y) - sat_white.AbsolutePosition).X / sat_white.AbsoluteSize.X,
                                0,
                                1
                        )
                        v = 1
                                - math.clamp(
                                        (vec2(mouse.X, mouse.Y) - sat_black.AbsolutePosition).Y / sat_black.AbsoluteSize.Y,
                                        0,
                                        1
                                )
                elseif dragging_hue then
                        h = math.clamp(
                                (vec2(mouse.X, mouse.Y) - hue_inline.AbsolutePosition).X / hue_inline.AbsoluteSize.X,
                                0,
                                1
                        )
                elseif dragging_alpha then
                        a = math.clamp(
                                (vec2(mouse.X, mouse.Y) - alpha_inline.AbsolutePosition).X / alpha_inline.AbsoluteSize.X,
                                0,
                                1
                        )
                end

                cfg.set(nil, nil)
        end

        alpha_inline.MouseButton1Down:Connect(function()
                dragging_alpha = true
        end)
        alpha_inline.MouseButton1Up:Connect(function()
                dragging_alpha = false
        end)

        hue_inline.MouseButton1Down:Connect(function()
                dragging_hue = true
        end)
        hue_inline.MouseButton1Up:Connect(function()
                dragging_hue = false
        end)

        sat_inline.MouseButton1Down:Connect(function()
                dragging_sat = true
        end)
        sat_inline.MouseButton1Up:Connect(function()
                dragging_sat = false
        end)

        cfg.saved_color = hsv(h, s, v)
        local selected = normal
        flags[cfg.flag]["animation"] = "normal"

        local selected_btn = normal
        local selected_mode = "normal"

        rainbow.MouseButton1Down:Connect(function()
                selected_btn.BackgroundTransparency = 1
                selected_btn = rainbow
                selected_mode = "rainbow"
                rainbow.BackgroundTransparency = 0

                flags[cfg.flag]["animation"] = "rainbow"
                cfg.saved_color = hsv(h, s, v)
        end)

        fade_alpha.MouseButton1Down:Connect(function()
                selected_btn.BackgroundTransparency = 1
                selected_btn = fade_alpha
                selected_mode = "fade_alpha"
                fade_alpha.BackgroundTransparency = 0

                flags[cfg.flag]["animation"] = "fade_alpha"
                cfg.saved_color = hsv(h, s, v)
        end)

        fade.MouseButton1Down:Connect(function()
                selected_btn.BackgroundTransparency = 1
                selected_btn = fade
                selected_mode = "fade"
                fade.BackgroundTransparency = 0

                flags[cfg.flag]["animation"] = "fade"
                cfg.saved_color = hsv(h, s, v)
        end)

        normal.MouseButton1Down:Connect(function()
                selected_btn.BackgroundTransparency = 1
                selected_btn = normal
                selected_mode = "normal"
                normal.BackgroundTransparency = 0

                flags[cfg.flag]["animation"] = "normal"
                cfg.set(cfg.saved_color)
        end)

        uis.InputEnded:Connect(function(input)
                if input.UserInputType == Enum.UserInputType.MouseButton1 then
                        dragging_sat = false
                        dragging_hue = false
                        dragging_alpha = false
                end
        end)

        uis.InputChanged:Connect(function(input)
                if
                        (dragging_sat or dragging_hue or dragging_alpha)
                        and input.UserInputType == Enum.UserInputType.MouseMovement
                then
                        cfg.update_color()
                end
        end)

        cfg.set(cfg.color, cfg.alpha)

        self.previous_holder = parent

        library.config_flags[cfg.flag] = cfg.set

        cfg.object = object

        task.spawn(function()
                while true do
                        if selected_mode ~= "normal" then
                                cfg.set(
                                        hsv(
                                                selected_mode == "rainbow" and library.sin or h,
                                                selected_mode == "rainbow" and 1 or s,
                                                selected_mode == "fade" and library.sin or v
                                        ),
                                        selected_mode == "fade_alpha" and library.sin
                                )
                        end
                        task.wait(0.1)
                end
        end)

        return setmetatable(cfg, library)
end

function library:keybind(properties)
        local cfg = {
                flag = properties.flag or tostring(2 ^ math.random(1, 30) * 3),
                keybind_name = properties.keybind_name or properties.displayName or properties.display or properties.name or "unknown",
                callback = properties.callback or function() end,
                open = false,
                binding = nil,
                name = properties.name or nil,
                key = properties.default or properties.key or nil,
                mode = properties.mode or "toggle",
                active = properties.active or false,
                display = properties.displayName or properties.display or properties.name or nil,
                hold_instances = {},
        }

        flags[cfg.flag] = {}

        local key
        if properties.no_list then
                key = {
                        set_visible = function() end,
                        change_text = function() end,
                        update = function() end,
                }
        else
                key = library:new_keybind({
                        text = cfg.display,
                        key = cfg.key,
                        mode = cfg.mode,
                })
        end

        local object
        local right_components
        if cfg.name then
                object = library:create("TextLabel", {
                        Parent = self.holder,
                        Name = "",
                        FontFace = library.font,
                        TextColor3 = Color3.fromRGB(170, 170, 170),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Text = cfg.name,
                        TextStrokeTransparency = 0.5,
                        Size = UDim2.new(1, -26, 0, 12),
                        BorderSizePixel = 0,
                        BackgroundTransparency = 1,
                        TextXAlignment = Enum.TextXAlignment.Left,
                        TextYAlignment = Enum.TextYAlignment.Top,
                        TextSize = 12,
                        BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                })

                right_components = library:create("Frame", {
                        Parent = object,
                        Name = "",
                        Position = UDim2.new(1, 15, 0, 1),
                        BorderColor3 = Color3.fromRGB(0, 0, 0),
                        Size = UDim2.new(0, 0, 1, 0),
                        BorderSizePixel = 0,
                        BackgroundColor3 = Color3.fromRGB(255, 255, 255),
                })

                local list = library:create("UIListLayout", {
                        Parent = right_components,
                        Name = "",
                        FillDirection = Enum.FillDirection.Horizontal,
                        HorizontalAlignment = Enum.HorizontalAlignment.Right,
                        Padding = UDim.new(0, 3),
                        SortOrder = Enum.SortOrder.LayoutOrder,
                })
        end

        local keybind = library:create("TextButton", {
                Parent = cfg.name and right_components or self.right_holder,
                Name = "",
                FontFace = library.font,
                TextColor3 = Color3.fromRGB(170, 170, 170),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Text = "ERROR",
                TextStrokeTransparency = 0.5,
                Size = UDim2.new(0, 16, 1, 0),
                BackgroundTransparency = 1,
                BorderSizePixel = 0,
                AutoButtonColor = false,
                TextSize = 12,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local content_inline = library:create("Frame", {
                Parent = library.gui,
                Name = "",
                BorderColor3 = Color3.fromRGB(19, 19, 19),
                Size = UDim2.new(0, 57, 0, 0),
                Position = dim2(0, keybind.AbsolutePosition.X, 0, keybind.AbsolutePosition.Y - 5),
                BorderSizePixel = 0,
                ZIndex = 2,
                AutomaticSize = Enum.AutomaticSize.Y,
                Visible = false,
                BackgroundColor3 = Color3.fromRGB(8, 8, 8),
        })

        keybind:GetPropertyChangedSignal("AbsolutePosition"):Connect(function()
                content_inline.Position = UDim2.new(0, keybind.AbsolutePosition.X, 0, keybind.AbsolutePosition.Y + 15)
        end)

        local content = library:create("Frame", {
                Parent = content_inline,
                Name = "",
                Position = UDim2.new(0, 2, 0, 2),
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Size = UDim2.new(1, -4, 1, -4),
                BackgroundColor3 = Color3.fromRGB(38, 38, 38),
        })

        local options = library:create("Frame", {
                Parent = content,
                Name = "",
                BackgroundTransparency = 1,
                Position = UDim2.new(0, 2, 0, 2),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Size = UDim2.new(1, -4, 1, -4),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(50, 50, 50),
        })

        local UIListLayout = library:create("UIListLayout", {
                Parent = options,
                Name = "",
                Padding = UDim.new(0, 2),
                SortOrder = Enum.SortOrder.LayoutOrder,
        })

        local press = library:create("TextButton", {
                Parent = options,
                Name = "",
                FontFace = library.font,
                TextColor3 = Color3.fromRGB(170, 170, 170),
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Text = "press",
                TextStrokeTransparency = 0.5,
                Size = UDim2.new(1, 0, 0, 12),
                BackgroundTransparency = 1,
                TextXAlignment = Enum.TextXAlignment.Left,
                Position = UDim2.new(0, 2, 0, 2),
                BorderSizePixel = 0,
                TextSize = 12,
                BackgroundColor3 = Color3.fromRGB(65, 65, 65),
        })

        local UIPadding = library:create("UIPadding", {
                Parent = press,
                Name = "",
                PaddingBottom = UDim.new(0, 1),
                PaddingLeft = UDim.new(0, 5),
        })

        local hold = library:create("TextButton", {
                Parent = options,
                Name = "",
                FontFace = library.font,
                TextColor3 = Color3.fromRGB(170, 170, 170),
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Text = "hold",
                TextStrokeTransparency = 0.5,
                Size = UDim2.new(1, 0, 0, 12),
                BackgroundTransparency = 1,
                TextXAlignment = Enum.TextXAlignment.Left,
                Position = UDim2.new(0, 2, 0, 2),
                BorderSizePixel = 0,
                TextSize = 12,
                BackgroundColor3 = Color3.fromRGB(65, 65, 65),
        })

        local UIPadding = library:create("UIPadding", {
                Parent = hold,
                Name = "",
                PaddingBottom = UDim.new(0, 1),
                PaddingLeft = UDim.new(0, 5),
        })

        local always = library:create("TextButton", {
                Parent = options,
                Name = "",
                FontFace = library.font,
                TextColor3 = Color3.fromRGB(170, 170, 170),
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Text = "always",
                TextStrokeTransparency = 0.5,
                Size = UDim2.new(1, 0, 0, 12),
                BackgroundTransparency = 1,
                TextXAlignment = Enum.TextXAlignment.Left,
                Position = UDim2.new(0, 2, 0, 2),
                BorderSizePixel = 0,
                TextSize = 12,
                BackgroundColor3 = Color3.fromRGB(65, 65, 65),
        })

        local UIPadding = library:create("UIPadding", {
                Parent = always,
                Name = "",
                PaddingBottom = UDim.new(0, 1),
                PaddingLeft = UDim.new(0, 5),
        })

        local UIPadding = library:create("UIPadding", {
                Parent = options,
                Name = "",
                PaddingBottom = UDim.new(0, 4),
        })

        function cfg.set_visible(bool)
                content_inline.Visible = bool

                if bool then
                        if library.current_element_open and library.current_element_open ~= cfg then
                                library.current_element_open.set_visible(false)
                                library.current_element_open.open = false
                        end

                        library.current_element_open = cfg
                end
        end

        function cfg.set_mode(mode)
                cfg.mode = mode

                if mode == "always" then
                        cfg.set(true)
                elseif mode == "hold" then
                        cfg.set(false)
                end

                flags[cfg.flag] = {
                        mode = cfg.mode,
                        key = cfg.key,
                        active = cfg.active,
                }

                flags[cfg.flag]["mode"] = mode
        end

        function cfg.set(input)
                if type(input) == "boolean" then
                        local __cached = input

                        if cfg.mode == "always" then
                                __cached = true
                        end

                        cfg.active = __cached
                        flags[cfg.flag]["active"] = __cached
                        cfg.callback(cfg.key, __cached)

                        flags[cfg.flag] = {
                                mode = cfg.mode,
                                key = cfg.key,
                                active = cfg.active,
                        }
                elseif tostring(input):find("Enum") then
                        input = input.Name == "Escape" and "..." or input

                        cfg.key = input or "..."

                        local _text = keys[cfg.key] or tostring(cfg.key):gsub("Enum.", "")
                        local _text2 = (tostring(_text):gsub("KeyCode.", ""):gsub("UserInputType.", "")) or "..."
                        cfg.key_name = _text2

                        flags[cfg.flag]["mode"] = cfg.mode
                        flags[cfg.flag]["key"] = cfg.key

                        keybind.Text = "[" .. string.lower(_text2) .. "]"

                        cfg.callback(cfg.key, cfg.active or false)

                        flags[cfg.flag] = {
                                mode = cfg.mode,
                                key = cfg.key,
                                active = cfg.active,
                        }
                elseif table.find({ "toggle", "hold", "always" }, input) then
                        cfg.set_mode(input)

                        if input == "always" then
                                cfg.active = true
                        end

                        cfg.callback(cfg.key, cfg.active or false)

                        flags[cfg.flag] = {
                                mode = cfg.mode,
                                key = cfg.key,
                                active = cfg.active,
                        }
                elseif type(input) == "table" then
                        input.key = type(input.key) == "string" and input.key ~= "..." and library:convert_enum(input.key)
                                or input.key

                        input.key = input.key == Enum.KeyCode.Escape and "..." or input.key
                        cfg.key = input.key or "..."

                        cfg.mode = input.mode or "toggle"

                        if input.active ~= nil then
                                cfg.active = input.active
                        end

                        flags[cfg.flag] = {
                                mode = cfg.mode,
                                key = cfg.key,
                                active = cfg.active,
                        }

                        local text = tostring(cfg.key) ~= "Enums" and (keys[cfg.key] or tostring(cfg.key):gsub("Enum.", "")) or nil
                        local __text = text and (tostring(text):gsub("KeyCode.", ""):gsub("UserInputType.", ""))

                        keybind.Text = "[" .. string.lower(__text) .. "]" or "..."
                        cfg.key_name = __text
                end

                if cfg.keybind_name then
                        key.change_text(keybind.Text .. " " .. cfg.keybind_name .. " (" .. flags[cfg.flag].mode .. ")")
                        key.set_visible(cfg.active)
                end
        end

        local selected

        hold.MouseButton1Click:Connect(function()
                if selected then
                        selected.BackgroundTransparency = 1
                end
                selected = hold
                hold.BackgroundTransparency = 0

                cfg.set_mode("hold")
                cfg.set_visible(false)
                cfg.open = false

                key.update({
                        text = cfg.display,
                        key = cfg.key,
                        mode = cfg.mode,
                })
        end)

        press.MouseButton1Click:Connect(function()
                if selected then
                        selected.BackgroundTransparency = 1
                end
                selected = press
                press.BackgroundTransparency = 0

                cfg.set_mode("toggle")
                cfg.set_visible(false)
                cfg.open = false

                key.update({
                        text = cfg.display,
                        key = cfg.key,
                        mode = cfg.mode,
                })
        end)

        always.MouseButton1Click:Connect(function()
                if selected then
                        selected.BackgroundTransparency = 1
                end
                selected = always

                always.BackgroundTransparency = 0
                cfg.set_mode("always")
                cfg.set_visible(false)
                cfg.open = false

                key.update({
                        text = cfg.display,
                        key = cfg.key,
                        mode = cfg.mode,
                })
        end)

        keybind.MouseButton2Click:Connect(function()
                cfg.open = not cfg.open

                cfg.set_visible(cfg.open)
        end)

        keybind.MouseButton1Down:Connect(function()
                task.wait()
                keybind.Text = "..."

                cfg.binding = library:connection(uis.InputBegan, function(input, game_event)
                        if input.UserInputType == Enum.UserInputType.Keyboard then
                                cfg.set(input.KeyCode)
                        elseif
                                input.UserInputType == Enum.UserInputType.MouseButton1
                                or Enum.UserInputType.MouseButton2
                                or Enum.UserInputType.MouseButton3
                        then

                                cfg.set(input.UserInputType)
                        end

                        key.update({
                                text = cfg.display,
                                key = cfg.key,
                                mode = cfg.mode,
                        })
                        cfg.binding:Disconnect()
                        cfg.binding = nil
                end)
        end)

        library:connection(uis.InputBegan, function(input, game_event)
                if not game_event then
                        if input.UserInputType == Enum.UserInputType.Keyboard then
                                if input.KeyCode == cfg.key then
                                        if cfg.mode == "toggle" then
                                                cfg.active = not cfg.active
                                                cfg.set(cfg.active)
                                        elseif cfg.mode == "hold" then
                                                cfg.set(true)
                                        end
                                end
                        else
                                if input.UserInputType == cfg.key then
                                        if cfg.mode == "toggle" then
                                                cfg.active = not cfg.active
                                                cfg.set(cfg.active)
                                        elseif cfg.mode == "hold" then
                                                cfg.set(true)
                                        end
                                end
                        end
                end
        end)

        library:connection(uis.InputEnded, function(input, game_event)
                if game_event then
                        return
                end

                local selected_key = input.UserInputType == Enum.UserInputType.Keyboard and input.KeyCode or input.UserInputType

                if selected_key == cfg.key then
                        if cfg.mode == "hold" then
                                cfg.set(false)
                        end
                end
        end)

        cfg.set({ mode = cfg.mode, active = cfg.active, key = cfg.key })
        key.update({
                text = cfg.display,
                key = cfg.key,
                mode = cfg.mode,
        })

        cfg.object = object

        library.config_flags[cfg.flag] = cfg.set

        return setmetatable(cfg, library)
end

function library:button(properties)
        local cfg = {
                callback = properties.callback or function() end,
                name = properties.text or properties.name or "Button",
        }

        local button_inline = library:create("Frame", {
                Parent = self.holder,
                Name = "",
                Position = UDim2.new(0, -15, 0, 2),
                BorderColor3 = Color3.fromRGB(19, 19, 19),
                Size = UDim2.new(1, -26, 0, 16),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(8, 8, 8),
        })
        library:apply_theme(button_inline, "contrast", "BackgroundColor3")

        local button = library:create("TextButton", {
                Parent = button_inline,
                Name = "",
                FontFace = library.font,
                TextColor3 = themes.preset.text,
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Text = cfg.name,
                TextStrokeTransparency = 0.5,
                Position = UDim2.new(0, 2, 0, 2),
                Size = UDim2.new(1, -4, 1, -4),
                TextSize = 12,
                AutoButtonColor = false,
                BackgroundColor3 = themes.preset.inline,
        })
        library:apply_theme(button, "text", "TextColor3")
        library:apply_theme(button, "inline", "BackgroundColor3")

        button.MouseButton1Click:Connect(function()
                cfg.callback()
        end)

        return setmetatable(cfg, library)
end

function library:textbox(properties)
        local cfg = {
                placeholder = properties.placeholder
                        or properties.placeholdertext
                        or properties.holder
                        or properties.holdertext
                        or "type here...",
                default = properties.default,
                clear_on_focus = properties.clearonfocus or false,
                flag = properties.flag or "...",
                callback = properties.callback or function() end,
        }

        local textbox_inline = library:create("Frame", {
                Parent = self.holder,
                Name = "",
                Position = UDim2.new(0, -15, 0, 2),
                BorderColor3 = Color3.fromRGB(19, 19, 19),
                Size = UDim2.new(1, -26, 0, 16),
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(8, 8, 8),
        })
        library:apply_theme(textbox_inline, "contrast", "BackgroundColor3")

        local textbox = library:create("TextBox", {
                Parent = textbox_inline,
                Name = "",
                FontFace = library.font,
                TextColor3 = Color3.fromRGB(170, 170, 170),
                BorderColor3 = Color3.fromRGB(56, 56, 56),
                Text = "",
                TextStrokeTransparency = 0.5,
                Position = UDim2.new(0, 2, 0, 2),
                Size = UDim2.new(1, -4, 1, -4),
                ClearTextOnFocus = cfg.clear_on_focus,
                PlaceholderColor3 = Color3.fromRGB(90, 90, 90),
                CursorPosition = -1,
                PlaceholderText = cfg.placeholder,
                TextSize = 12,
                BackgroundColor3 = Color3.fromRGB(38, 38, 38),
        })
        library:apply_theme(textbox, "text", "TextColor3")
        library:apply_theme(textbox, "unselected_text", "PlaceholderColor3")

        textbox:GetPropertyChangedSignal("Text"):Connect(function()
                flags[cfg.flag] = textbox.text
                cfg.callback(textbox.text)
        end)

        function cfg.set(text)
                flags[cfg.flag] = text
                textbox.Text = text
                cfg.callback(text)
        end

        if cfg.default then
                cfg.set(cfg.default)
        end

        library.config_flags[cfg.flag] = cfg.set

        return setmetatable(cfg, library)
end

local _teSliding = false
local _teSlTrack, _teSlFill, _teSlLbl, _teSlName, _teSlMin, _teSlMax, _teSlCb
local _teSVDrag = false
local _teSVBox2, _teSVMkr, _teSVCb
local _teHueDrag = false
local _teHueSl2, _teHueMk2, _teHueSVRef, _teHueCb
local _selectedRow = nil

function library:open_tool_explorer()
        if _teGui then
                _teGui.Enabled = true
                return
        end

        _teGui = Instance.new("ScreenGui")
        _teGui.Name = "ToolExplorerGui"
        _teGui.ResetOnSpawn = false
        _teGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
        _teGui.DisplayOrder = 998

        if gethui then
                _teGui.Parent = gethui()
        elseif syn and syn.protect_gui then
                syn.protect_gui(_teGui)
                _teGui.Parent = game:GetService("CoreGui")
        else
                _teGui.Parent = game:GetService("CoreGui")
        end

        -- central drag / input connections
        library:connection(uis.InputChanged, function(i)
                if i.UserInputType ~= Enum.UserInputType.MouseMovement then return end
                if _teSliding and _teSlTrack then
                        local pct = math.clamp((i.Position.X - _teSlTrack.AbsolutePosition.X) / _teSlTrack.AbsoluteSize.X, 0, 1)
                        if _teSlFill then _teSlFill.Size = UDim2.new(pct, 0, 1, 0) end
                        local val = _teSlMin + (_teSlMax - _teSlMin) * pct
                        if _teSlLbl then _teSlLbl.Text = (_teSlName or "") .. ":  " .. string.format("%.3g", val) end
                        if _teSlCb then pcall(_teSlCb, val) end
                end
                if _teSVDrag and _teSVBox2 then
                        local s2 = math.clamp((i.Position.X - _teSVBox2.AbsolutePosition.X) / _teSVBox2.AbsoluteSize.X, 0, 1)
                        local v2 = 1 - math.clamp((i.Position.Y - _teSVBox2.AbsolutePosition.Y) / _teSVBox2.AbsoluteSize.Y, 0, 1)
                        if _teSVMkr then _teSVMkr.Position = UDim2.new(s2, -2, 1 - v2, -2) end
                        if _teSVCb then pcall(_teSVCb, s2, v2) end
                end
                if _teHueDrag and _teHueSl2 then
                        local h2 = math.clamp((i.Position.Y - _teHueSl2.AbsolutePosition.Y) / _teHueSl2.AbsoluteSize.Y, 0, 1)
                        if _teHueMk2 then _teHueMk2.Position = UDim2.new(0, -2, h2, 0) end
                        if _teHueSVRef then _teHueSVRef.BackgroundColor3 = Color3.fromHSV(h2, 1, 1) end
                        if _teHueCb then pcall(_teHueCb, h2) end
                end
        end)

        library:connection(uis.InputEnded, function(i)
                if i.UserInputType == Enum.UserInputType.MouseButton1 then
                        _teSliding = false
                        _teSVDrag = false
                        _teHueDrag = false
                end
        end)

        local EW, EH = 565, 410
        local Main = library:create("Frame", {
                Name = "ToolExplorer",
                Size = UDim2.new(0, EW, 0, EH),
                Position = UDim2.new(0.5, -EW/2, 0.5, -EH/2),
                BackgroundColor3 = Color3.fromRGB(8, 8, 8),
                BorderSizePixel = 0,
                ZIndex = 100,
                Parent = _teGui,
        })

        library:create("UIStroke", {
                Parent = Main,
                Color = Color3.fromRGB(57, 57, 57),
                Thickness = 1,
                LineJoinMode = Enum.LineJoinMode.Miter,
        })

        local left_bar = library:create("Frame", {
                Size = UDim2.new(0, 2, 1, 0),
                BackgroundColor3 = themes.preset.accent,
                BorderSizePixel = 0,
                ZIndex = 105,
                Parent = Main,
        })
        library:apply_theme(left_bar, "accent", "BackgroundColor3")

        local TitleBar = library:create("Frame", {
                Size = UDim2.new(1, 0, 0, 28),
                BackgroundColor3 = Color3.fromRGB(13, 13, 13),
                BorderSizePixel = 0,
                ZIndex = 103,
                Parent = Main,
        })

        library:create("UIStroke", {
                Parent = TitleBar,
                Color = Color3.fromRGB(19, 19, 19),
                Thickness = 1,
                LineJoinMode = Enum.LineJoinMode.Miter,
        })

        local title_accent = library:create("Frame", {
                Size = UDim2.new(1, 0, 0, 1),
                Position = UDim2.new(0, 0, 1, -1),
                BackgroundColor3 = themes.preset.accent,
                BorderSizePixel = 0,
                ZIndex = 104,
                Parent = TitleBar,
        })
        library:apply_theme(title_accent, "accent", "BackgroundColor3")

        library:create("TextLabel", {
                Text = "invera.priv  /  Tool Explorer",
                Size = UDim2.new(1, -82, 1, 0),
                Position = UDim2.new(0, 8, 0, 0),
                BackgroundTransparency = 1,
                FontFace = library.font,
                TextSize = 12,
                TextColor3 = Color3.fromRGB(255, 255, 255),
                TextXAlignment = Enum.TextXAlignment.Left,
                ZIndex = 104,
                Parent = TitleBar,
                TextStrokeTransparency = 0,
                TextStrokeColor3 = Color3.fromRGB(0, 0, 0),
        })

        local RefreshBtn = library:create("TextButton", {
                Text = "refresh",
                Size = UDim2.new(0, 46, 0, 16),
                Position = UDim2.new(1, -72, 0, 6),
                BackgroundColor3 = Color3.fromRGB(26, 26, 26),
                BorderSizePixel = 0,
                FontFace = library.font,
                TextSize = 12,
                TextColor3 = themes.preset.unselected_text,
                AutoButtonColor = false,
                ZIndex = 104,
                Parent = TitleBar,
                TextStrokeTransparency = 0,
                TextStrokeColor3 = Color3.fromRGB(0, 0, 0),
        })

        library:create("UIStroke", {
                Parent = RefreshBtn,
                Color = Color3.fromRGB(19, 19, 19),
                Thickness = 1,
                LineJoinMode = Enum.LineJoinMode.Miter,
        })

        local CloseBtn = library:create("TextButton", {
                Text = "x",
                Size = UDim2.new(0, 22, 0, 16),
                Position = UDim2.new(1, -25, 0, 6),
                BackgroundColor3 = Color3.fromRGB(25, 25, 25),
                BorderSizePixel = 0,
                FontFace = library.font,
                TextSize = 12,
                TextColor3 = Color3.fromRGB(255, 255, 255),
                AutoButtonColor = false,
                ZIndex = 104,
                Parent = TitleBar,
                TextStrokeTransparency = 0,
                TextStrokeColor3 = Color3.fromRGB(0, 0, 0),
        })

        library:create("UIStroke", {
                Parent = CloseBtn,
                Color = Color3.fromRGB(19, 19, 19),
                Thickness = 1,
                LineJoinMode = Enum.LineJoinMode.Miter,
        })

        CloseBtn.MouseButton1Click:Connect(function()
                _teGui.Enabled = false
        end)

        -- title bar drag with lerp
        local _teDrag, _teDragStart, _teDragOrigin = false, nil, nil
        local _teDragTarget = Main.Position
        TitleBar.InputBegan:Connect(function(i)
                if i.UserInputType == Enum.UserInputType.MouseButton1 then
                        _teDrag = true
                        _teDragStart = i.Position
                        _teDragOrigin = Main.Position
                        _teDragTarget = Main.Position
                end
        end)
        library:connection(uis.InputChanged, function(i)
                if _teDrag and i.UserInputType == Enum.UserInputType.MouseMovement then
                        local d = i.Position - _teDragStart
                        _teDragTarget = UDim2.new(_teDragOrigin.X.Scale, _teDragOrigin.X.Offset + d.X, _teDragOrigin.Y.Scale, _teDragOrigin.Y.Offset + d.Y)
                end
        end)
        library:connection(uis.InputEnded, function(i)
                if i.UserInputType == Enum.UserInputType.MouseButton1 then
                        _teDrag = false
                end
        end)
        library:connection(game:GetService("RunService").Heartbeat, function(dt)
                if _teDrag then
                        Main.Position = Main.Position:Lerp(_teDragTarget, math.clamp(dt * 15, 0, 1))
                end
        end)

        local Body = library:create("Frame", {
                Size = UDim2.new(1, 0, 1, -46),
                Position = UDim2.new(0, 0, 0, 28),
                BackgroundTransparency = 1,
                ZIndex = 102,
                Parent = Main,
        })

        -- Left Panel
        local LeftPanel = library:create("ScrollingFrame", {
                Size = UDim2.new(0, 186, 1, -4),
                Position = UDim2.new(0, 4, 0, 2),
                BackgroundColor3 = Color3.fromRGB(15, 15, 15),
                BorderSizePixel = 0,
                ScrollBarThickness = 3,
                ScrollBarImageColor3 = themes.preset.accent,
                AutomaticCanvasSize = Enum.AutomaticSize.Y,
                CanvasSize = UDim2.new(0, 0, 0, 0),
                ZIndex = 103,
                Parent = Body,
        })
        library:apply_theme(LeftPanel, "accent", "ScrollBarImageColor3")

        library:create("UIStroke", {
                Parent = LeftPanel,
                Color = Color3.fromRGB(19, 19, 19),
                Thickness = 1,
                LineJoinMode = Enum.LineJoinMode.Miter,
        })

        library:create("UIListLayout", {
                SortOrder = Enum.SortOrder.LayoutOrder,
                Padding = UDim.new(0, 1),
                Parent = LeftPanel,
        })
        library:create("UIPadding", {
                PaddingTop = UDim.new(0, 4),
                PaddingBottom = UDim.new(0, 4),
                PaddingLeft = UDim.new(0, 4),
                PaddingRight = UDim.new(0, 4),
                Parent = LeftPanel,
        })

        -- Separator
        library:create("Frame", {
                Size = UDim2.new(0, 1, 1, -4),
                Position = UDim2.new(0, 192, 0, 2),
                BackgroundColor3 = Color3.fromRGB(19, 19, 19),
                BorderSizePixel = 0,
                ZIndex = 103,
                Parent = Body,
        })

        -- Right Panel
        local RightPanel = library:create("ScrollingFrame", {
                Size = UDim2.new(1, -199, 1, -4),
                Position = UDim2.new(0, 195, 0, 2),
                BackgroundColor3 = Color3.fromRGB(15, 15, 15),
                BorderSizePixel = 0,
                ScrollBarThickness = 3,
                ScrollBarImageColor3 = themes.preset.accent,
                AutomaticCanvasSize = Enum.AutomaticSize.Y,
                CanvasSize = UDim2.new(0, 0, 0, 0),
                ZIndex = 103,
                Parent = Body,
        })
        library:apply_theme(RightPanel, "accent", "ScrollBarImageColor3")

        library:create("UIStroke", {
                Parent = RightPanel,
                Color = Color3.fromRGB(19, 19, 19),
                Thickness = 1,
                LineJoinMode = Enum.LineJoinMode.Miter,
        })

        library:create("UIListLayout", {
                SortOrder = Enum.SortOrder.LayoutOrder,
                Padding = UDim.new(0, 4),
                Parent = RightPanel,
        })
        library:create("UIPadding", {
                PaddingTop = UDim.new(0, 6),
                PaddingBottom = UDim.new(0, 10),
                PaddingLeft = UDim.new(0, 7),
                PaddingRight = UDim.new(0, 7),
                Parent = RightPanel,
        })

        -- Bottom bar
        local BotBar = library:create("Frame", {
                Size = UDim2.new(1, 0, 0, 18),
                Position = UDim2.new(0, 0, 1, -18),
                BackgroundColor3 = Color3.fromRGB(11, 11, 11),
                BorderSizePixel = 0,
                ZIndex = 103,
                Parent = Main,
        })
        library:create("Frame", {
                Size = UDim2.new(1, 0, 0, 1),
                BackgroundColor3 = Color3.fromRGB(28, 28, 28),
                BorderSizePixel = 0,
                ZIndex = 104,
                Parent = BotBar,
        })
        library:create("TextLabel", {
                Text = "invera.priv  |  Tool Explorer  |  click a part to edit",
                Size = UDim2.new(1, -10, 1, 0),
                Position = UDim2.new(0, 8, 0, 0),
                BackgroundTransparency = 1,
                FontFace = library.font,
                TextSize = 9,
                TextColor3 = themes.preset.unselected_text,
                TextXAlignment = Enum.TextXAlignment.Left,
                ZIndex = 104,
                Parent = BotBar,
                TextStrokeTransparency = 0,
                TextStrokeColor3 = Color3.fromRGB(0, 0, 0),
        })

        -- Helpers
        local function IsWithin(frame, input)
                local pos = frame.AbsolutePosition
                local size = frame.AbsoluteSize
                local mx, my = input.Position.X, input.Position.Y
                return mx >= pos.X and mx <= pos.X + size.X and my >= pos.Y and my <= pos.Y + size.Y
        end

        local function ClearRP()
                for _, c in pairs(RightPanel:GetChildren()) do
                        if not c:IsA("UIListLayout") and not c:IsA("UIPadding") then c:Destroy() end
                end
        end

        local function RPLbl(text, color, h)
                return library:create("TextLabel", {
                        Text = text,
                        Size = UDim2.new(1, 0, 0, h or 14),
                        BackgroundTransparency = 1,
                        FontFace = library.font,
                        TextSize = 12,
                        TextColor3 = color or themes.preset.unselected_text,
                        TextXAlignment = Enum.TextXAlignment.Left,
                        ZIndex = 104,
                        Parent = RightPanel,
                        TextStrokeTransparency = 0,
                        TextStrokeColor3 = Color3.fromRGB(0, 0, 0),
                })
        end

        local function RPDiv()
                library:create("Frame", {
                        Size = UDim2.new(1, 0, 0, 1),
                        BackgroundColor3 = Color3.fromRGB(19, 19, 19),
                        BorderSizePixel = 0,
                        ZIndex = 104,
                        Parent = RightPanel,
                })
        end

        -- =========================================================
        -- WIDGET HELPERS  (styled to match main UI library)
        -- =========================================================
        local _rpEffects  = {}
        local _rpSpinning = {}
        local _rpLightFx  = {}
        local _rpRgbFx    = {}

        local function RPSection(name, col)
                local hr = library:create("Frame", {
                        Size = UDim2.new(1,0,0,20),
                        BackgroundColor3 = Color3.fromRGB(13,13,13),
                        BorderSizePixel = 0,
                        ZIndex = 104,
                        Parent = RightPanel,
                })
                library:create("UIStroke",{Parent=hr,Color=Color3.fromRGB(19,19,19),Thickness=1,LineJoinMode=Enum.LineJoinMode.Miter})
                local ac = library:create("Frame",{Size=UDim2.new(1,0,0,1),Position=UDim2.new(0,0,1,-1),BackgroundColor3=col or themes.preset.accent,BorderSizePixel=0,ZIndex=105,Parent=hr})
                if not col then library:apply_theme(ac,"accent","BackgroundColor3") end
                local lbl = library:create("TextLabel",{Text=name:upper(),Size=UDim2.new(1,-8,1,0),Position=UDim2.new(0,8,0,0),BackgroundTransparency=1,FontFace=library.font,TextSize=10,TextColor3=col or themes.preset.accent,TextXAlignment=Enum.TextXAlignment.Left,ZIndex=105,Parent=hr,TextStrokeTransparency=0,TextStrokeColor3=Color3.fromRGB(0,0,0)})
                if not col then library:apply_theme(lbl,"accent","TextColor3") end
        end

        local function RPToggle(name, default, cb)
                local enabled = default or false
                local holder = library:create("Frame",{Size=UDim2.new(1,0,0,14),BackgroundTransparency=1,ZIndex=104,Parent=RightPanel})
                local ii = library:create("TextButton",{Size=UDim2.new(0,10,0,10),Position=UDim2.new(0,0,0,2),BackgroundColor3=themes.preset.contrast,BorderSizePixel=0,Text="",AutoButtonColor=false,ZIndex=105,Parent=holder})
                library:create("UIStroke",{Parent=ii,Color=Color3.fromRGB(38,38,38),Thickness=1,LineJoinMode=Enum.LineJoinMode.Miter})
                local icon = library:create("Frame",{Size=UDim2.new(1,-4,1,-4),Position=UDim2.new(0,2,0,2),BackgroundColor3=themes.preset.bg,BorderSizePixel=0,ZIndex=106,Parent=ii})
                local dot = library:create("Frame",{Size=UDim2.new(1,0,1,0),BackgroundColor3=themes.preset.accent,BorderSizePixel=0,ZIndex=107,Visible=enabled,Parent=icon})
                library:apply_theme(dot,"accent","BackgroundColor3")
                library:create("TextLabel",{Text=name,Size=UDim2.new(1,-16,1,0),Position=UDim2.new(0,14,0,0),BackgroundTransparency=1,FontFace=library.font,TextSize=12,TextColor3=themes.preset.text,TextXAlignment=Enum.TextXAlignment.Left,ZIndex=105,Parent=holder,TextStrokeTransparency=0,TextStrokeColor3=Color3.fromRGB(0,0,0)})
                local btn = library:create("TextButton",{Size=UDim2.new(1,0,1,0),BackgroundTransparency=1,Text="",ZIndex=108,Parent=holder})
                btn.MouseButton1Click:Connect(function()
                        enabled = not enabled
                        dot.Visible = enabled
                        if cb then pcall(cb, enabled) end
                end)
                local ctrl = {}
                ctrl.setEnabled = function(v) enabled = v; dot.Visible = v end
                return ctrl
        end

        local function RPSlider(name, min, max, cur, cb)
                local holder = library:create("Frame", {
                        Size = UDim2.new(1, 0, 0, 32),
                        BackgroundTransparency = 1,
                        ZIndex = 104,
                        Parent = RightPanel,
                })
                local lbl = library:create("TextLabel", {
                        Text = name .. ":  " .. string.format("%.3g", cur),
                        Size = UDim2.new(1, -26, 0, 12),
                        BackgroundTransparency = 1,
                        FontFace = library.font,
                        TextSize = 12,
                        TextColor3 = themes.preset.text,
                        TextXAlignment = Enum.TextXAlignment.Left,
                        ZIndex = 105,
                        Parent = holder,
                        TextStrokeTransparency = 0,
                        TextStrokeColor3 = Color3.fromRGB(0, 0, 0),
                })
                -- sub/add buttons matching main UI -/+
                local sub = library:create("TextButton", {
                        Text = "-",
                        Size = UDim2.new(0, 8, 0, 8),
                        Position = UDim2.new(0, 0, 0, 13),
                        BackgroundColor3 = themes.preset.contrast,
                        BorderSizePixel = 0,
                        FontFace = library.font,
                        TextSize = 10,
                        TextColor3 = themes.preset.unselected_text,
                        AutoButtonColor = false,
                        ZIndex = 106,
                        Parent = holder,
                })
                library:create("UIStroke", {Parent = sub, Color = Color3.fromRGB(19,19,19), Thickness = 1, LineJoinMode = Enum.LineJoinMode.Miter})
                -- slider_inline: contrast outer (main UI style)
                local slider_inline = library:create("Frame", {
                        Size = UDim2.new(1, -22, 0, 8),
                        Position = UDim2.new(0, 12, 0, 14),
                        BackgroundColor3 = themes.preset.contrast,
                        BorderSizePixel = 0,
                        ZIndex = 105,
                        Parent = holder,
                })
                library:create("UIStroke", {Parent = slider_inline, Color = Color3.fromRGB(19,19,19), Thickness = 1, LineJoinMode = Enum.LineJoinMode.Miter})
                -- slider_bg: bg inner
                local slider_bg = library:create("Frame", {
                        Size = UDim2.new(1,-4,1,-4),
                        Position = UDim2.new(0,2,0,2),
                        BackgroundColor3 = themes.preset.bg,
                        BorderSizePixel = 0,
                        ZIndex = 106,
                        Parent = slider_inline,
                })
                -- fill_inline: progress bar
                local fill_inline = library:create("Frame", {
                        Size = UDim2.new(math.clamp((cur - min) / math.max(max - min, 0.0001), 0, 1), 0, 1, 0),
                        BackgroundColor3 = themes.preset.inline,
                        BorderSizePixel = 0,
                        ZIndex = 107,
                        Parent = slider_bg,
                })
                -- fill: accent color bar
                local fill = library:create("Frame", {
                        Size = UDim2.new(1,0,1,-4),
                        Position = UDim2.new(0,0,0,2),
                        BackgroundColor3 = themes.preset.accent,
                        BorderSizePixel = 0,
                        ZIndex = 108,
                        Parent = fill_inline,
                })
                library:apply_theme(fill, "accent", "BackgroundColor3")
                -- add button
                local add = library:create("TextButton", {
                        Text = "+",
                        Size = UDim2.new(0, 8, 0, 8),
                        Position = UDim2.new(1, -8, 0, 13),
                        BackgroundColor3 = themes.preset.contrast,
                        BorderSizePixel = 0,
                        FontFace = library.font,
                        TextSize = 10,
                        TextColor3 = themes.preset.unselected_text,
                        AutoButtonColor = false,
                        ZIndex = 106,
                        Parent = holder,
                })
                library:create("UIStroke", {Parent = add, Color = Color3.fromRGB(19,19,19), Thickness = 1, LineJoinMode = Enum.LineJoinMode.Miter})
                -- click/drag overlay
                local tBtn = library:create("TextButton", {
                        Size = UDim2.new(1,0,1,0),
                        BackgroundTransparency = 1,
                        Text = "",
                        ZIndex = 109,
                        Parent = slider_inline,
                })
                local function applyVal(pct)
                        pct = math.clamp(pct, 0, 1)
                        fill_inline.Size = UDim2.new(pct, 0, 1, 0)
                        local val = min + (max - min) * pct
                        lbl.Text = name .. ":  " .. string.format("%.3g", val)
                        if cb then pcall(cb, val) end
                end
                tBtn.InputBegan:Connect(function(i)
                        if i.UserInputType == Enum.UserInputType.MouseButton1 then
                                _teSliding = true
                                _teSlTrack = slider_bg
                                _teSlFill = fill_inline
                                _teSlLbl = lbl
                                _teSlName = name
                                _teSlMin = min
                                _teSlMax = max
                                _teSlCb = cb
                                local pct = math.clamp((i.Position.X - slider_bg.AbsolutePosition.X) / slider_bg.AbsoluteSize.X, 0, 1)
                                applyVal(pct)
                        end
                end)
                tBtn.InputEnded:Connect(function(i)
                        if i.UserInputType == Enum.UserInputType.MouseButton1 then _teSliding = false end
                end)
                local step = (max - min) / 50
                sub.MouseButton1Click:Connect(function()
                        applyVal(fill_inline.Size.X.Scale - step / (max - min))
                end)
                add.MouseButton1Click:Connect(function()
                        applyVal(fill_inline.Size.X.Scale + step / (max - min))
                end)
        end

        local function RPColorPicker(name, defaultColor, cb)
                local _h, _s, _v = Color3.toHSV(defaultColor)
                local holder = library:create("Frame", {
                        Size = UDim2.new(1,0,0,14),
                        BackgroundTransparency = 1,
                        ZIndex = 104,
                        Parent = RightPanel,
                })
                library:create("TextLabel", {
                        Text = name,
                        Size = UDim2.new(1,-24,1,0),
                        BackgroundTransparency = 1,
                        FontFace = library.font,
                        TextSize = 12,
                        TextColor3 = themes.preset.text,
                        TextXAlignment = Enum.TextXAlignment.Left,
                        ZIndex = 105,
                        Parent = holder,
                        TextStrokeTransparency = 0,
                        TextStrokeColor3 = Color3.fromRGB(0,0,0),
                })
                -- color swatch: contrast outer + color inner (matches main UI)
                local icon_inline = library:create("Frame", {
                        Size = UDim2.new(0,18,0,10),
                        Position = UDim2.new(1,-18,0,2),
                        BackgroundColor3 = themes.preset.contrast,
                        BorderSizePixel = 0,
                        ZIndex = 105,
                        Parent = holder,
                })
                library:create("UIStroke", {Parent=icon_inline, Color=Color3.fromRGB(19,19,19), Thickness=1, LineJoinMode=Enum.LineJoinMode.Miter})
                local icon = library:create("Frame", {
                        Size = UDim2.new(1,-4,1,-4),
                        Position = UDim2.new(0,2,0,2),
                        BackgroundColor3 = defaultColor,
                        BorderSizePixel = 0,
                        ZIndex = 106,
                        Parent = icon_inline,
                })
                local CPBtn = library:create("TextButton", {
                        Size = UDim2.new(1,0,1,0),
                        BackgroundTransparency = 1,
                        Text = "",
                        ZIndex = 107,
                        Parent = icon_inline,
                })
                -- Picker popup parented to _teGui
                local Picker = library:create("Frame", {
                        Size = UDim2.new(0,160,0,163),
                        BackgroundColor3 = Color3.fromRGB(8,8,8),
                        Visible = false,
                        ZIndex = 6000,
                        Parent = _teGui,
                })
                library:create("UIStroke", {Parent=Picker, Color=Color3.fromRGB(57,57,57), Thickness=1, LineJoinMode=Enum.LineJoinMode.Miter})
                local SVBox = library:create("Frame", {
                        Size = UDim2.new(0,120,0,120),
                        Position = UDim2.new(0,10,0,10),
                        BackgroundColor3 = Color3.fromHSV(_h,1,1),
                        ZIndex = 6001,
                        Parent = Picker,
                })
                library:create("UIStroke", {Parent=SVBox, Color=Color3.fromRGB(19,19,19), Thickness=1, LineJoinMode=Enum.LineJoinMode.Miter})
                local SatGrad = library:create("Frame", {Size=UDim2.new(1,0,1,0), BackgroundColor3=Color3.new(1,1,1), ZIndex=6002, Parent=SVBox})
                library:create("UIGradient", {Color=ColorSequence.new(Color3.new(1,1,1)), Transparency=NumberSequence.new(0,1), Parent=SatGrad})
                local ValGrad = library:create("Frame", {Size=UDim2.new(1,0,1,0), BackgroundColor3=Color3.new(0,0,0), ZIndex=6003, Parent=SVBox})
                library:create("UIGradient", {Rotation=90, Color=ColorSequence.new(Color3.new(0,0,0)), Transparency=NumberSequence.new(1,0), Parent=ValGrad})
                local Mkr = library:create("Frame", {Size=UDim2.new(0,4,0,4), Position=UDim2.new(_s,-2,1-_v,-2), BackgroundColor3=Color3.new(1,1,1), ZIndex=6004, Parent=SVBox})
                library:create("UIStroke", {Color=Color3.new(0,0,0), Thickness=1, Parent=Mkr})
                local SVBtn = library:create("TextButton", {Size=UDim2.new(1,0,1,0), BackgroundTransparency=1, Text="", ZIndex=6005, Parent=SVBox})
                local HueSl = library:create("Frame", {Size=UDim2.new(0,12,0,120), Position=UDim2.new(1,-22,0,10), ZIndex=6001, Parent=Picker})
                library:create("UIStroke", {Parent=HueSl, Color=Color3.fromRGB(19,19,19), Thickness=1, LineJoinMode=Enum.LineJoinMode.Miter})
                library:create("UIGradient", {
                        Rotation=90,
                        Color=ColorSequence.new({
                                ColorSequenceKeypoint.new(0,   Color3.fromRGB(255,0,0)),
                                ColorSequenceKeypoint.new(0.167,Color3.fromRGB(255,255,0)),
                                ColorSequenceKeypoint.new(0.333,Color3.fromRGB(0,255,0)),
                                ColorSequenceKeypoint.new(0.5,  Color3.fromRGB(0,255,255)),
                                ColorSequenceKeypoint.new(0.667,Color3.fromRGB(0,0,255)),
                                ColorSequenceKeypoint.new(0.833,Color3.fromRGB(255,0,255)),
                                ColorSequenceKeypoint.new(1,    Color3.fromRGB(255,0,0)),
                        }),
                        Parent=HueSl,
                })
                local HueMk = library:create("Frame", {Size=UDim2.new(1,4,0,2), Position=UDim2.new(0,-2,_h,0), BackgroundColor3=Color3.new(1,1,1), ZIndex=6002, Parent=HueSl})
                local HueBtn = library:create("TextButton", {Size=UDim2.new(1,0,1,0), BackgroundTransparency=1, Text="", ZIndex=6005, Parent=HueSl})
                -- Hex row
                local HexFrame = library:create("Frame", {Size=UDim2.new(0,118,0,14), Position=UDim2.new(0,10,0,136), BackgroundColor3=Color3.fromRGB(26,26,26), BorderSizePixel=0, ZIndex=6005, Parent=Picker})
                library:create("UIStroke", {Parent=HexFrame, Color=Color3.fromRGB(19,19,19), Thickness=1, LineJoinMode=Enum.LineJoinMode.Miter})
                library:create("TextLabel", {Text="#", Size=UDim2.new(0,10,1,0), Position=UDim2.new(0,3,0,0), BackgroundTransparency=1, FontFace=library.font, TextSize=10, TextColor3=themes.preset.unselected_text, TextXAlignment=Enum.TextXAlignment.Left, ZIndex=6006, Parent=HexFrame, TextStrokeTransparency=0, TextStrokeColor3=Color3.fromRGB(0,0,0)})
                local HexInput = library:create("TextBox", {Text=string.format("%02X%02X%02X",math.floor(defaultColor.R*255+0.5),math.floor(defaultColor.G*255+0.5),math.floor(defaultColor.B*255+0.5)), PlaceholderText="RRGGBB", Size=UDim2.new(1,-16,1,0), Position=UDim2.new(0,14,0,0), BackgroundTransparency=1, FontFace=library.font, TextSize=10, TextColor3=Color3.fromRGB(170,170,170), TextXAlignment=Enum.TextXAlignment.Left, ClearTextOnFocus=false, ZIndex=6006, Parent=HexFrame, TextStrokeTransparency=0, TextStrokeColor3=Color3.fromRGB(0,0,0)})

                local function toHex(c)
                        return string.format("%02X%02X%02X",math.floor(c.R*255+0.5),math.floor(c.G*255+0.5),math.floor(c.B*255+0.5))
                end
                local function ClosePicker()
                        Picker.Visible = false
                        _teSVDrag = false
                        _teHueDrag = false
                end
                CPBtn.MouseButton1Click:Connect(function()
                        if Picker.Visible then ClosePicker() return end
                        local pos = icon_inline.AbsolutePosition
                        Picker.Position = UDim2.new(0,pos.X+24,0,pos.Y)
                        Picker.Visible = true
                end)
                SVBtn.InputBegan:Connect(function(i)
                        if i.UserInputType == Enum.UserInputType.MouseButton1 then
                                _teSVDrag = true
                                _teSVBox2 = SVBox
                                _teSVMkr = Mkr
                                _teSVCb = function(ns, nv)
                                        _s = ns; _v = nv
                                        local c = Color3.fromHSV(_h,_s,_v)
                                        icon.BackgroundColor3 = c
                                        HexInput.Text = toHex(c)
                                        if cb then pcall(cb, c) end
                                end
                        end
                end)
                SVBtn.InputEnded:Connect(function(i)
                        if i.UserInputType == Enum.UserInputType.MouseButton1 then _teSVDrag = false end
                end)
                HueBtn.InputBegan:Connect(function(i)
                        if i.UserInputType == Enum.UserInputType.MouseButton1 then
                                _teHueDrag = true
                                _teHueSl2 = HueSl
                                _teHueMk2 = HueMk
                                _teHueSVRef = SVBox
                                _teHueCb = function(nh)
                                        _h = nh
                                        SVBox.BackgroundColor3 = Color3.fromHSV(_h,1,1)
                                        local c = Color3.fromHSV(_h,_s,_v)
                                        icon.BackgroundColor3 = c
                                        HexInput.Text = toHex(c)
                                        if cb then pcall(cb, c) end
                                end
                        end
                end)
                HueBtn.InputEnded:Connect(function(i)
                        if i.UserInputType == Enum.UserInputType.MouseButton1 then _teHueDrag = false end
                end)
                HexInput.FocusLost:Connect(function()
                        local txt = HexInput.Text:gsub("[^%x]","")
                        if #txt == 6 then
                                local ok2, c = pcall(Color3.fromHex, txt)
                                if ok2 then
                                        _h,_s,_v = c:ToHSV()
                                        HueMk.Position = UDim2.new(0,-2,_h,0)
                                        SVBox.BackgroundColor3 = Color3.fromHSV(_h,1,1)
                                        Mkr.Position = UDim2.new(_s,-2,1-_v,-2)
                                        icon.BackgroundColor3 = c
                                        if cb then pcall(cb, c) end
                                end
                        end
                        HexInput.Text = toHex(icon.BackgroundColor3)
                end)
                library:connection(uis.InputBegan, function(inp)
                        if inp.UserInputType == Enum.UserInputType.MouseButton1 and Picker.Visible then
                                if not IsWithin(Picker,inp) and not IsWithin(icon_inline,inp) then ClosePicker() end
                        end
                end)
        end

        local function RPDropdown(name, options, default, cb)
                local holder = library:create("Frame", {
                        Size = UDim2.new(1,0,0,34),
                        BackgroundTransparency = 1,
                        ZIndex = 104,
                        Parent = RightPanel,
                })
                library:create("TextLabel", {
                        Text = name,
                        Size = UDim2.new(1,0,0,12),
                        BackgroundTransparency = 1,
                        FontFace = library.font,
                        TextSize = 12,
                        TextColor3 = themes.preset.text,
                        TextXAlignment = Enum.TextXAlignment.Left,
                        ZIndex = 105,
                        Parent = holder,
                        TextStrokeTransparency = 0,
                        TextStrokeColor3 = Color3.fromRGB(0,0,0),
                })
                local selected = default or options[1] or ""
                -- dropdown_inline: contrast outer (main UI style)
                local dropdown_inline = library:create("Frame", {
                        Size = UDim2.new(1,0,0,16),
                        Position = UDim2.new(0,0,0,14),
                        BackgroundColor3 = themes.preset.contrast,
                        BorderSizePixel = 0,
                        ZIndex = 105,
                        Parent = holder,
                })
                library:create("UIStroke", {Parent=dropdown_inline, Color=Color3.fromRGB(19,19,19), Thickness=1, LineJoinMode=Enum.LineJoinMode.Miter})
                -- dropdown: inner bg
                local dropdown = library:create("TextButton", {
                        Text = "",
                        Size = UDim2.new(1,-4,1,-4),
                        Position = UDim2.new(0,2,0,2),
                        BackgroundColor3 = Color3.fromRGB(38,38,38),
                        BorderSizePixel = 0,
                        ZIndex = 106,
                        Parent = dropdown_inline,
                })
                local ValueLabel = library:create("TextLabel", {
                        Text = selected,
                        Size = UDim2.new(1,-18,1,0),
                        Position = UDim2.new(0,4,0,0),
                        BackgroundTransparency = 1,
                        FontFace = library.font,
                        TextSize = 12,
                        TextColor3 = themes.preset.text,
                        TextXAlignment = Enum.TextXAlignment.Left,
                        ZIndex = 107,
                        Parent = dropdown,
                        TextStrokeTransparency = 0,
                        TextStrokeColor3 = Color3.fromRGB(0,0,0),
                })
                local icon_lbl = library:create("TextLabel", {
                        Text = "+",
                        Size = UDim2.new(0,14,1,0),
                        Position = UDim2.new(1,-14,0,0),
                        BackgroundTransparency = 1,
                        FontFace = library.font,
                        TextSize = 8,
                        TextColor3 = themes.preset.unselected_text,
                        ZIndex = 107,
                        Parent = dropdown,
                })
                local DropContainer = library:create("Frame", {
                        BackgroundColor3 = themes.preset.contrast,
                        BorderSizePixel = 0,
                        Visible = false,
                        ZIndex = 7000,
                        Parent = _teGui,
                })
                library:create("UIStroke", {Parent=DropContainer, Color=Color3.fromRGB(57,57,57), Thickness=1, LineJoinMode=Enum.LineJoinMode.Miter})
                local dropped = false
                local function UpdateList()
                        DropContainer:ClearAllChildren()
                        library:create("UIListLayout", {SortOrder=Enum.SortOrder.LayoutOrder, Parent=DropContainer})
                        DropContainer.Size = UDim2.new(0, dropdown_inline.AbsoluteSize.X, 0, math.clamp(#options*16,16,200))
                        for _, opt in ipairs(options) do
                                local ob = library:create("TextButton", {
                                        Text = "  " .. opt,
                                        Size = UDim2.new(1,0,0,16),
                                        BackgroundColor3 = (opt==selected) and Color3.fromRGB(38,38,38) or Color3.fromRGB(26,26,26),
                                        FontFace = library.font,
                                        TextSize = 12,
                                        TextColor3 = (opt==selected) and themes.preset.accent or themes.preset.text,
                                        TextXAlignment = Enum.TextXAlignment.Left,
                                        ZIndex = 7001,
                                        Parent = DropContainer,
                                        TextStrokeTransparency = 0,
                                        TextStrokeColor3 = Color3.fromRGB(0,0,0),
                                })
                                ob.MouseButton1Click:Connect(function()
                                        selected = opt
                                        ValueLabel.Text = opt
                                        dropped = false
                                        DropContainer.Visible = false
                                        icon_lbl.Text = "+"
                                        if cb then pcall(cb, opt) end
                                end)
                        end
                end
                dropdown.MouseButton1Click:Connect(function()
                        dropped = not dropped
                        icon_lbl.Text = dropped and "-" or "+"
                        if dropped then
                                task.defer(function()
                                        UpdateList()
                                        DropContainer.Position = UDim2.new(0,dropdown_inline.AbsolutePosition.X,0,dropdown_inline.AbsolutePosition.Y+18)
                                        DropContainer.Visible = true
                                end)
                        else
                                DropContainer.Visible = false
                        end
                end)
                library:connection(uis.InputBegan, function(inp)
                        if inp.UserInputType == Enum.UserInputType.MouseButton1 and dropped then
                                if not IsWithin(DropContainer,inp) and not IsWithin(dropdown_inline,inp) then
                                        dropped = false
                                        DropContainer.Visible = false
                                        icon_lbl.Text = "+"
                                end
                        end
                end)
        end

        local function RPTextBox(name, default, cb)
                local holder = library:create("Frame", {
                        Size = UDim2.new(1,0,0,34),
                        BackgroundTransparency = 1,
                        ZIndex = 104,
                        Parent = RightPanel,
                })
                library:create("TextLabel", {
                        Text = name,
                        Size = UDim2.new(1,0,0,14),
                        BackgroundTransparency = 1,
                        FontFace = library.font,
                        TextSize = 12,
                        TextColor3 = themes.preset.text,
                        TextXAlignment = Enum.TextXAlignment.Left,
                        ZIndex = 105,
                        Parent = holder,
                        TextStrokeTransparency = 0,
                        TextStrokeColor3 = Color3.fromRGB(0,0,0),
                })
                -- contrast outer
                local box_inline = library:create("Frame", {
                        Size = UDim2.new(1,0,0,16),
                        Position = UDim2.new(0,0,0,16),
                        BackgroundColor3 = themes.preset.contrast,
                        BorderSizePixel = 0,
                        ZIndex = 105,
                        Parent = holder,
                })
                library:create("UIStroke", {Parent=box_inline, Color=Color3.fromRGB(19,19,19), Thickness=1, LineJoinMode=Enum.LineJoinMode.Miter})
                local box_bg = library:create("Frame", {
                        Size = UDim2.new(1,-4,1,-4),
                        Position = UDim2.new(0,2,0,2),
                        BackgroundColor3 = Color3.fromRGB(38,38,38),
                        BorderSizePixel = 0,
                        ZIndex = 106,
                        Parent = box_inline,
                })
                local Input = library:create("TextBox", {
                        Text = tostring(default or ""),
                        PlaceholderText = "...",
                        Size = UDim2.new(1,-8,1,0),
                        Position = UDim2.new(0,4,0,0),
                        BackgroundTransparency = 1,
                        FontFace = library.font,
                        TextSize = 12,
                        TextColor3 = themes.preset.text,
                        TextXAlignment = Enum.TextXAlignment.Left,
                        ClearTextOnFocus = false,
                        ZIndex = 107,
                        Parent = box_bg,
                        TextStrokeTransparency = 0,
                        TextStrokeColor3 = Color3.fromRGB(0,0,0),
                })
                Input.FocusLost:Connect(function()
                        if cb then pcall(cb, Input.Text) end
                end)
        end

        local function RPButton(label, cb, tcolor)
                local holder = library:create("Frame", {
                        Size = UDim2.new(1,0,0,18),
                        BackgroundTransparency = 1,
                        ZIndex = 104,
                        Parent = RightPanel,
                })
                local btn_inline = library:create("Frame", {
                        Size = UDim2.new(1,0,1,0),
                        BackgroundColor3 = themes.preset.contrast,
                        BorderSizePixel = 0,
                        ZIndex = 105,
                        Parent = holder,
                })
                library:create("UIStroke", {Parent=btn_inline, Color=Color3.fromRGB(19,19,19), Thickness=1, LineJoinMode=Enum.LineJoinMode.Miter})
                local btn = library:create("TextButton", {
                        Text = label,
                        Size = UDim2.new(1,-4,1,-4),
                        Position = UDim2.new(0,2,0,2),
                        BackgroundColor3 = Color3.fromRGB(38,38,38),
                        BorderSizePixel = 0,
                        FontFace = library.font,
                        TextSize = 12,
                        TextColor3 = tcolor or themes.preset.text,
                        AutoButtonColor = false,
                        ZIndex = 106,
                        Parent = btn_inline,
                        TextStrokeTransparency = 0,
                        TextStrokeColor3 = Color3.fromRGB(0,0,0),
                })
                btn.MouseEnter:Connect(function() btn.BackgroundColor3 = Color3.fromRGB(50,50,50) end)
                btn.MouseLeave:Connect(function() btn.BackgroundColor3 = Color3.fromRGB(38,38,38) end)
                btn.MouseButton1Click:Connect(function()
                        btn.BackgroundColor3 = Color3.fromRGB(60,60,60)
                        task.delay(0.1, function() btn.BackgroundColor3 = Color3.fromRGB(38,38,38) end)
                        if cb then pcall(cb) end
                end)
        end

        -- ================================================================
        -- OVERRIDE SYSTEM
        -- ================================================================
        local function RecordOverride(obj, prop, value)
                local tool = obj:FindFirstAncestorOfClass("Tool")
                if tool then
                        library._toolExplorerOverrides = library._toolExplorerOverrides or {}
                        library._toolExplorerOverrides[tool.Name] = library._toolExplorerOverrides[tool.Name] or {}
                        local relativePath = {}
                        local cur = obj
                        while cur and cur ~= tool do
                                table.insert(relativePath, 1, cur.Name)
                                cur = cur.Parent
                        end
                        local pathStr = table.concat(relativePath, "/")
                        library._toolExplorerOverrides[tool.Name][pathStr] = library._toolExplorerOverrides[tool.Name][pathStr] or {}
                        library._toolExplorerOverrides[tool.Name][pathStr][prop] = value
                end
        end

        local function ApplyOverridesToTool(tool)
                if not library._toolExplorerOverrides then return end
                local overrides = library._toolExplorerOverrides[tool.Name]
                if overrides then
                        for pathStr, props in pairs(overrides) do
                                local target
                                if pathStr == "" then
                                        target = tool
                                else
                                        local parts = string.split(pathStr, "/")
                                        target = tool
                                        for _, partName in ipairs(parts) do
                                                if target then target = target:FindFirstChild(partName) end
                                        end
                                end
                                if target then
                                        for prop, val in pairs(props) do
                                                pcall(function() target[prop] = val end)
                                        end
                                end
                        end
                end
        end

        if not library._watcherInit then
                library._watcherInit = true
                library._toolExplorerOverrides = library._toolExplorerOverrides or {}
                task.spawn(function()
                        local lp2 = game:GetService("Players").LocalPlayer
                        local function watchChar(char)
                                if not char then return end
                                char.ChildAdded:Connect(function(child)
                                        if child:IsA("Tool") then task.wait(0.1); ApplyOverridesToTool(child) end
                                end)
                                for _, child in ipairs(char:GetChildren()) do
                                        if child:IsA("Tool") then ApplyOverridesToTool(child) end
                                end
                        end
                        lp2.Backpack.ChildAdded:Connect(function(child)
                                if child:IsA("Tool") then task.wait(0.1); ApplyOverridesToTool(child) end
                        end)
                        for _, child in ipairs(lp2.Backpack:GetChildren()) do
                                if child:IsA("Tool") then ApplyOverridesToTool(child) end
                        end
                        lp2.CharacterAdded:Connect(watchChar)
                        if lp2.Character then watchChar(lp2.Character) end
                end)
        end

        -- ================================================================
        -- BUILD PROPERTIES  (6 categories: Parts, Meshes, Particles,
        --                    Lights, Effects, Explorer UI)
        -- ================================================================
        local function BuildProperties(obj)
                ClearRP()
                local ok, cname = pcall(function() return obj.ClassName end)
                if not ok then RPLbl("Could not read object.") return end

                -- Object header
                local hdr = library:create("Frame", {
                        Size = UDim2.new(1,0,0,24),
                        BackgroundColor3 = Color3.fromRGB(13,13,13),
                        BorderSizePixel = 0,
                        ZIndex = 104,
                        Parent = RightPanel,
                })
                library:create("UIStroke", {Parent=hdr, Color=Color3.fromRGB(19,19,19), Thickness=1, LineJoinMode=Enum.LineJoinMode.Miter})
                local hdrAc = library:create("Frame", {Size=UDim2.new(1,0,0,1), Position=UDim2.new(0,0,1,-1), BackgroundColor3=themes.preset.accent, BorderSizePixel=0, ZIndex=105, Parent=hdr})
                library:apply_theme(hdrAc,"accent","BackgroundColor3")
                local hdrLbl = library:create("TextLabel", {
                        Text = obj.Name.."  ["..cname.."]",
                        Size = UDim2.new(1,-8,1,0),
                        Position = UDim2.new(0,8,0,0),
                        BackgroundTransparency = 1,
                        FontFace = library.font,
                        TextSize = 11,
                        TextColor3 = themes.preset.accent,
                        TextXAlignment = Enum.TextXAlignment.Left,
                        ZIndex = 105,
                        Parent = hdr,
                        TextStrokeTransparency = 0,
                        TextStrokeColor3 = Color3.fromRGB(0,0,0),
                })
                library:apply_theme(hdrLbl,"accent","TextColor3")

                RPTextBox("Name", obj.Name, function(t)
                        pcall(function() obj.Name = t end)
                        RecordOverride(obj, "Name", t)
                        hdrLbl.Text = t.."  ["..cname.."]"
                end)

                -- ==============  PARTS  ==============
                if obj:IsA("BasePart") then
                        RPSection("Parts")
                        local allowedMats = {
                                "Plastic","SmoothPlastic","Wood","WoodPlanks","Brick","Cobblestone",
                                "Concrete","Granite","Marble","Slate","Pebble","Sand","Fabric",
                                "Grass","LeafyGrass","Ground","Mud","Snow","Ice","Glacier","Rock",
                                "Sandstone","Limestone","Basalt","Asphalt","Pavement","Salt",
                                "CrackedLava","Metal","DiamondPlate","CorrodedMetal","Foil",
                                "Glass","Neon","ForceField",
                        }
                        local rawMat = tostring(obj.Material):match("%.(.+)$") or "SmoothPlastic"
                        local curMat = "SmoothPlastic"
                        for _, m in ipairs(allowedMats) do if rawMat == m then curMat = m break end end

                        RPDropdown("Material", allowedMats, curMat, function(vv)
                                pcall(function() obj.Material = Enum.Material[vv] end)
                                RecordOverride(obj, "Material", Enum.Material[vv])
                                task.defer(function() BuildProperties(obj) end)
                        end)
                        RPColorPicker("Color", obj.Color, function(c)
                                pcall(function() obj.Color = c end)
                                RecordOverride(obj, "Color", c)
                        end)
                        RPToggle("RGB Color", false, function(on)
                                if _rpRgbFx[obj] then _rpRgbFx[obj] = false end
                                if on then
                                        _rpRgbFx[obj] = true
                                        task.spawn(function()
                                                while _rpRgbFx[obj] do
                                                        pcall(function() obj.Color = Color3.fromHSV((tick()%3)/3,1,1) end)
                                                        task.wait(0.05)
                                                end
                                        end)
                                end
                        end)
                        if curMat == "Neon" then
                                RPSection("Neon / HDR", Color3.fromRGB(255,200,0))
                                library._originalColorScale = library._originalColorScale or {}
                                RPSlider("HDR Intensity", 1, 20, 1, function(vv)
                                        pcall(function()
                                                library._originalColorScale[obj] = library._originalColorScale[obj] or obj.Color
                                                local orig = library._originalColorScale[obj]
                                                obj.Color = Color3.new(math.clamp(orig.R*vv,0,1),math.clamp(orig.G*vv,0,1),math.clamp(orig.B*vv,0,1))
                                        end)
                                end)
                        end
                        RPSlider("Transparency", 0, 1, obj.Transparency, function(vv)
                                pcall(function() obj.Transparency = vv end)
                                RecordOverride(obj, "Transparency", vv)
                        end)
                        RPSlider("Reflectance", 0, 1, obj.Reflectance, function(vv)
                                pcall(function() obj.Reflectance = vv end)
                                RecordOverride(obj, "Reflectance", vv)
                        end)
                        RPSection("Resize")
                        local sz = obj.Size
                        RPSlider("Size X", 0.1, 50, sz.X, function(vv)
                                pcall(function() obj.Size = Vector3.new(vv,obj.Size.Y,obj.Size.Z) end)
                                RecordOverride(obj,"Size",obj.Size)
                        end)
                        RPSlider("Size Y", 0.1, 50, sz.Y, function(vv)
                                pcall(function() obj.Size = Vector3.new(obj.Size.X,vv,obj.Size.Z) end)
                                RecordOverride(obj,"Size",obj.Size)
                        end)
                        RPSlider("Size Z", 0.1, 50, sz.Z, function(vv)
                                pcall(function() obj.Size = Vector3.new(obj.Size.X,obj.Size.Y,vv) end)
                                RecordOverride(obj,"Size",obj.Size)
                        end)
                        RPSection("Rotate")
                        local spinAxisPart = "Y"
                        local spinSpeedPart = 30
                        RPDropdown("Spin Axis", {"X","Y","Z"}, "Y", function(vv) spinAxisPart = vv end)
                        RPSlider("Spin Speed", 1, 200, 30, function(vv) spinSpeedPart = vv end)
                        RPToggle("Spin Part", false, function(on)
                                if _rpSpinning[obj] then _rpSpinning[obj]:Disconnect(); _rpSpinning[obj] = nil end
                                if on then
                                        _rpSpinning[obj] = run_service.Heartbeat:Connect(function(dt)
                                                pcall(function()
                                                        if obj and obj.Parent then
                                                                local rad = math.rad(spinSpeedPart * dt)
                                                                if spinAxisPart=="X" then obj.CFrame = obj.CFrame*CFrame.Angles(rad,0,0)
                                                                elseif spinAxisPart=="Y" then obj.CFrame = obj.CFrame*CFrame.Angles(0,rad,0)
                                                                else obj.CFrame = obj.CFrame*CFrame.Angles(0,0,rad) end
                                                        else
                                                                if _rpSpinning[obj] then _rpSpinning[obj]:Disconnect(); _rpSpinning[obj]=nil end
                                                        end
                                                end)
                                        end)
                                end
                        end)
                        RPSection("Texture / Decal")
                        local texChild = nil
                        local decalChild = nil
                        pcall(function()
                                for _, c in pairs(obj:GetChildren()) do
                                        if c:IsA("Texture") and not texChild then texChild = c end
                                        if c:IsA("Decal") and not decalChild then decalChild = c end
                                end
                        end)
                        RPTextBox("Texture ID (add/set)", texChild and texChild.Texture or "", function(t)
                                pcall(function()
                                        local tx = obj:FindFirstChildOfClass("Texture") or Instance.new("Texture",obj)
                                        tx.Texture = t; tx.Face = Enum.NormalId.Front
                                        RecordOverride(tx,"Texture",t)
                                end)
                        end)
                        RPTextBox("Decal ID (add/set)", decalChild and decalChild.Texture or "", function(t)
                                pcall(function()
                                        local dc = obj:FindFirstChildOfClass("Decal") or Instance.new("Decal",obj)
                                        dc.Texture = t; dc.Face = Enum.NormalId.Front
                                        RecordOverride(dc,"Texture",t)
                                end)
                        end)
                        library._textureScroll = library._textureScroll or {}
                        library._textureSpin = library._textureSpin or {}
                        library._scrollers = library._scrollers or {}
                        local function updateTexAnim(tc)
                                if library._scrollers[tc] then library._scrollers[tc]:Disconnect(); library._scrollers[tc]=nil end
                                local scroll = library._textureScroll[tc] or 0
                                local spin = library._textureSpin[tc] or 0
                                if scroll > 0 or spin > 0 then
                                        library._scrollers[tc] = run_service.RenderStepped:Connect(function(dt)
                                                pcall(function()
                                                        if tc and tc.Parent then
                                                                if scroll>0 then tc.OffsetStudsU=(tc.OffsetStudsU+scroll*dt*2)%100 end
                                                                if spin>0 then tc.Rotation=(tc.Rotation+spin*dt*25)%360 end
                                                        else
                                                                if library._scrollers[tc] then library._scrollers[tc]:Disconnect();library._scrollers[tc]=nil end
                                                        end
                                                end)
                                        end)
                                end
                        end
                        if texChild then
                                RPSlider("Texture Scroll Speed",0,20,library._textureScroll[texChild] or 0,function(vv)
                                        library._textureScroll[texChild]=vv; updateTexAnim(texChild)
                                end)
                                RPSlider("Texture Spin Speed",0,20,library._textureSpin[texChild] or 0,function(vv)
                                        library._textureSpin[texChild]=vv; updateTexAnim(texChild)
                                end)
                        end
                        RPSection("Part Actions")
                        RPDropdown("Cast Shadow",{"true","false"},obj.CastShadow and "true" or "false",function(vv)
                                pcall(function() obj.CastShadow=vv=="true" end)
                                RecordOverride(obj,"CastShadow",vv=="true")
                        end)
                        RPDropdown("Can Collide",{"true","false"},obj.CanCollide and "true" or "false",function(vv)
                                pcall(function() obj.CanCollide=vv=="true" end)
                                RecordOverride(obj,"CanCollide",vv=="true")
                        end)
                        RPDropdown("Anchored",{"true","false"},obj.Anchored and "true" or "false",function(vv)
                                pcall(function() obj.Anchored=vv=="true" end)
                                RecordOverride(obj,"Anchored",vv=="true")
                        end)
                        RPButton("Duplicate Part", function()
                                pcall(function()
                                        local clone = obj:Clone()
                                        clone.Parent = obj.Parent
                                        clone.CFrame = obj.CFrame*CFrame.new(obj.Size.X+0.5,0,0)
                                end)
                        end)
                        RPButton("Apply Color+Mat To All Parts", function()
                                pcall(function()
                                        local tool = obj:FindFirstAncestorOfClass("Tool")
                                        if tool then
                                                for _, d in pairs(tool:GetDescendants()) do
                                                        if d:IsA("BasePart") then d.Color=obj.Color; d.Material=obj.Material end
                                                end
                                        end
                                end)
                        end)
                        RPButton("Reset Part", function()
                                pcall(function() obj.Transparency=0; obj.Reflectance=0; obj.CastShadow=true; obj.CanCollide=true end)
                        end)
                        RPButton("Delete Part", function()
                                pcall(function() obj:Destroy() end)
                                ClearRP(); RPLbl("Part deleted.", themes.preset.unselected_text)
                        end, Color3.fromRGB(220,80,80))
                end

                -- ==============  MESHES  ==============
                if obj:IsA("MeshPart") then
                        RPSection("Meshes")
                        RPTextBox("Mesh ID", obj.MeshId, function(t)
                                pcall(function() obj.MeshId=t end)
                                RecordOverride(obj,"MeshId",t)
                        end)
                        RPTextBox("Texture ID", obj.TextureID, function(t)
                                pcall(function() obj.TextureID=t end)
                                RecordOverride(obj,"TextureID",t)
                        end)
                        RPColorPicker("Vertex Color", obj.Color, function(c)
                                pcall(function() obj.Color=c end)
                                RecordOverride(obj,"Color",c)
                        end)
                        RPToggle("RGB Mesh", false, function(on)
                                local key="mesh_"..tostring(obj)
                                if _rpRgbFx[key] then _rpRgbFx[key]=false end
                                if on then
                                        _rpRgbFx[key]=true
                                        task.spawn(function()
                                                while _rpRgbFx[key] do
                                                        pcall(function() obj.Color=Color3.fromHSV((tick()%3)/3,1,1) end)
                                                        task.wait(0.05)
                                                end
                                        end)
                                end
                        end)
                        RPToggle("Neon Mesh", false, function(on)
                                pcall(function() obj.Material=on and Enum.Material.Neon or Enum.Material.SmoothPlastic end)
                        end)
                        RPSlider("Transparency",0,1,obj.Transparency,function(vv)
                                pcall(function() obj.Transparency=vv end); RecordOverride(obj,"Transparency",vv)
                        end)
                        RPSlider("Reflectance",0,1,obj.Reflectance,function(vv)
                                pcall(function() obj.Reflectance=vv end); RecordOverride(obj,"Reflectance",vv)
                        end)
                        RPDropdown("Material",{"Plastic","SmoothPlastic","Neon","Glass","ForceField","Metal","DiamondPlate"},tostring(obj.Material):match("%.(.+)$") or "SmoothPlastic",function(vv)
                                pcall(function() obj.Material=Enum.Material[vv] end)
                                RecordOverride(obj,"Material",Enum.Material[vv])
                        end)
                        RPDropdown("DoubleSided",{"true","false"},obj.DoubleSided and "true" or "false",function(vv)
                                pcall(function() obj.DoubleSided=vv=="true" end)
                        end)
                        RPButton("Remove Texture", function() pcall(function() obj.TextureID="" end); RecordOverride(obj,"TextureID","") end)
                        RPButton("Reset Mesh", function()
                                pcall(function() obj.Material=Enum.Material.SmoothPlastic; obj.Transparency=0; obj.Reflectance=0 end)
                        end)
                end

                if obj:IsA("SpecialMesh") then
                        RPSection("Meshes")
                        RPTextBox("Mesh ID", obj.MeshId, function(t)
                                pcall(function() obj.MeshId=t end); RecordOverride(obj,"MeshId",t)
                        end)
                        RPTextBox("Texture ID", obj.TextureId, function(t)
                                pcall(function() obj.TextureId=t end); RecordOverride(obj,"TextureId",t)
                        end)
                        RPDropdown("Mesh Type",{"FileMesh","Sphere","Cylinder","Brick","Head","Torso","Wedge"},tostring(obj.MeshType):match("%.(.+)$") or "FileMesh",function(vv)
                                pcall(function() obj.MeshType=Enum.MeshType[vv] end)
                        end)
                        RPSection("Mesh Scale")
                        local sc=obj.Scale
                        RPSlider("Scale X",0,20,sc.X,function(vv) pcall(function() obj.Scale=Vector3.new(vv,obj.Scale.Y,obj.Scale.Z) end) end)
                        RPSlider("Scale Y",0,20,sc.Y,function(vv) pcall(function() obj.Scale=Vector3.new(obj.Scale.X,vv,obj.Scale.Z) end) end)
                        RPSlider("Scale Z",0,20,sc.Z,function(vv) pcall(function() obj.Scale=Vector3.new(obj.Scale.X,obj.Scale.Y,vv) end) end)
                        RPSection("Mesh Offset")
                        local off=obj.Offset
                        RPSlider("Offset X",-10,10,off.X,function(vv) pcall(function() obj.Offset=Vector3.new(vv,obj.Offset.Y,obj.Offset.Z) end) end)
                        RPSlider("Offset Y",-10,10,off.Y,function(vv) pcall(function() obj.Offset=Vector3.new(obj.Offset.X,vv,obj.Offset.Z) end) end)
                        RPSlider("Offset Z",-10,10,off.Z,function(vv) pcall(function() obj.Offset=Vector3.new(obj.Offset.X,obj.Offset.Y,vv) end) end)
                        RPButton("Remove Texture", function() pcall(function() obj.TextureId="" end) end)
                        RPButton("Reset Mesh", function() pcall(function() obj.Scale=Vector3.new(1,1,1); obj.Offset=Vector3.zero end) end)
                end

                -- ==============  DECALS / TEXTURES  ==============
                if obj:IsA("Decal") or obj:IsA("Texture") then
                        RPSection("Texture / Decal  ["..cname.."]")
                        RPTextBox("Texture ID", obj.Texture, function(t)
                                pcall(function() obj.Texture=t end); RecordOverride(obj,"Texture",t)
                        end)
                        RPColorPicker("Color Tint", obj.Color3, function(c)
                                pcall(function() obj.Color3=c end); RecordOverride(obj,"Color3",c)
                        end)
                        RPSlider("Transparency",0,1,obj.Transparency,function(vv)
                                pcall(function() obj.Transparency=vv end); RecordOverride(obj,"Transparency",vv)
                        end)
                        RPDropdown("Face",{"Top","Bottom","Left","Right","Front","Back"},tostring(obj.Face):match("%.(.+)$") or "Front",function(vv)
                                pcall(function() obj.Face=Enum.NormalId[vv] end); RecordOverride(obj,"Face",Enum.NormalId[vv])
                        end)
                        if obj:IsA("Texture") then
                                RPSlider("Studs Per Tile U",0.1,50,obj.StudsPerTileU,function(vv)
                                        pcall(function() obj.StudsPerTileU=vv end); RecordOverride(obj,"StudsPerTileU",vv)
                                end)
                                RPSlider("Studs Per Tile V",0.1,50,obj.StudsPerTileV,function(vv)
                                        pcall(function() obj.StudsPerTileV=vv end); RecordOverride(obj,"StudsPerTileV",vv)
                                end)
                                library._textureScroll = library._textureScroll or {}
                                library._textureSpin = library._textureSpin or {}
                                library._scrollers = library._scrollers or {}
                                local function updateAnim2()
                                        if library._scrollers[obj] then library._scrollers[obj]:Disconnect(); library._scrollers[obj]=nil end
                                        local scroll=library._textureScroll[obj] or 0
                                        local spin=library._textureSpin[obj] or 0
                                        if scroll>0 or spin>0 then
                                                library._scrollers[obj]=run_service.RenderStepped:Connect(function(dt)
                                                        pcall(function()
                                                                if obj and obj.Parent then
                                                                        if scroll>0 then obj.OffsetStudsU=(obj.OffsetStudsU+scroll*dt*2)%100 end
                                                                        if spin>0 then obj.Rotation=(obj.Rotation+spin*dt*25)%360 end
                                                                else
                                                                        if library._scrollers[obj] then library._scrollers[obj]:Disconnect();library._scrollers[obj]=nil end
                                                                end
                                                        end)
                                                end)
                                        end
                                end
                                RPSlider("Texture Scroll Speed",0,20,library._textureScroll[obj] or 0,function(vv)
                                        library._textureScroll[obj]=vv; updateAnim2()
                                end)
                                RPSlider("Texture Spin Speed",0,20,library._textureSpin[obj] or 0,function(vv)
                                        library._textureSpin[obj]=vv; updateAnim2()
                                end)
                        end
                        RPSlider("Z-Index",1,10,obj.ZIndex,function(vv)
                                pcall(function() obj.ZIndex=vv end); RecordOverride(obj,"ZIndex",vv)
                        end)
                end

                -- ==============  PARTICLES  ==============
                if obj:IsA("ParticleEmitter") then
                        RPSection("Particles")
                        RPToggle("Enabled", obj.Enabled, function(on)
                                pcall(function() obj.Enabled=on end); RecordOverride(obj,"Enabled",on)
                        end)
                        RPTextBox("Texture ID", obj.Texture, function(t)
                                pcall(function() obj.Texture=t end); RecordOverride(obj,"Texture",t)
                        end)
                        local startC=Color3.new(1,1,1)
                        pcall(function() startC=obj.Color.Keypoints[1].Value end)
                        RPColorPicker("Color", startC, function(c)
                                pcall(function() obj.Color=ColorSequence.new(c) end)
                                RecordOverride(obj,"Color",ColorSequence.new(c))
                        end)
                        RPToggle("RGB Particles", false, function(on)
                                local key="p_"..tostring(obj)
                                if _rpRgbFx[key] then _rpRgbFx[key]=false end
                                if on then
                                        _rpRgbFx[key]=true
                                        task.spawn(function()
                                                while _rpRgbFx[key] do
                                                        pcall(function() obj.Color=ColorSequence.new(Color3.fromHSV((tick()%3)/3,1,1)) end)
                                                        task.wait(0.05)
                                                end
                                        end)
                                end
                        end)
                        RPSlider("Rate (p/sec)",0,1000,obj.Rate,function(vv)
                                pcall(function() obj.Rate=vv end); RecordOverride(obj,"Rate",vv)
                        end)
                        local lifMin=1; local lifMax=5
                        pcall(function() lifMin=obj.Lifetime.Min; lifMax=obj.Lifetime.Max end)
                        RPSlider("Lifetime Min",0.1,50,lifMin,function(vv)
                                pcall(function() obj.Lifetime=NumberRange.new(vv,math.max(vv,obj.Lifetime.Max)) end)
                        end)
                        RPSlider("Lifetime Max",0.1,50,lifMax,function(vv)
                                pcall(function() obj.Lifetime=NumberRange.new(math.min(vv,obj.Lifetime.Min),vv) end)
                        end)
                        local spdMin=10; pcall(function() spdMin=obj.Speed.Min end)
                        RPSlider("Speed",0,500,spdMin,function(vv)
                                pcall(function() obj.Speed=NumberRange.new(vv,vv) end)
                        end)
                        RPSlider("Drag",0,20,obj.Drag,function(vv)
                                pcall(function() obj.Drag=vv end); RecordOverride(obj,"Drag",vv)
                        end)
                        local sprd=10; pcall(function() sprd=obj.SpreadAngle.X end)
                        RPSlider("Spread Angle",0,180,sprd,function(vv)
                                pcall(function() obj.SpreadAngle=Vector2.new(vv,vv) end)
                        end)
                        local rot2=0; pcall(function() rot2=obj.Rotation.Min end)
                        RPSlider("Rotation",-360,360,rot2,function(vv)
                                pcall(function() obj.Rotation=NumberRange.new(vv,vv) end)
                        end)
                        RPSlider("Rotation Speed",-1000,1000,obj.RotSpeed.Min,function(vv)
                                pcall(function() obj.RotSpeed=NumberRange.new(vv,vv) end)
                        end)
                        RPSlider("Light Emission",0,1,obj.LightEmission,function(vv)
                                pcall(function() obj.LightEmission=vv end); RecordOverride(obj,"LightEmission",vv)
                        end)
                        RPSlider("Light Influence",0,1,obj.LightInfluence,function(vv)
                                pcall(function() obj.LightInfluence=vv end)
                        end)
                        RPSlider("Z-Offset",-10,10,obj.ZOffset,function(vv)
                                pcall(function() obj.ZOffset=vv end)
                        end)
                        RPSlider("Time Scale",0,5,obj.TimeScale,function(vv)
                                pcall(function() obj.TimeScale=vv end)
                        end)
                        RPDropdown("Emit Direction",{"Top","Bottom","Front","Back","Left","Right"},tostring(obj.EmissionDirection):match("%.(.+)$") or "Top",function(vv)
                                pcall(function() obj.EmissionDirection=Enum.NormalId[vv] end)
                        end)
                        RPButton("Burst (x50)", function() pcall(function() obj:Emit(50) end) end)
                        local par=obj.Parent
                        if par then
                                RPButton("+ Trail", function()
                                        pcall(function()
                                                if not par:FindFirstChildOfClass("Trail") then
                                                        local a0=Instance.new("Attachment"); a0.Name="TrailA"; a0.Parent=par
                                                        local a1=Instance.new("Attachment"); a1.Name="TrailB"; a1.CFrame=CFrame.new(0,2,0); a1.Parent=par
                                                        local tr=Instance.new("Trail"); tr.Attachment0=a0; tr.Attachment1=a1; tr.Parent=par
                                                end
                                        end)
                                end)
                                RPButton("+ Fire", function() pcall(function() if not par:FindFirstChildOfClass("Fire") then Instance.new("Fire").Parent=par end end) end)
                                RPButton("+ Smoke", function() pcall(function() if not par:FindFirstChildOfClass("Smoke") then Instance.new("Smoke").Parent=par end end) end)
                                RPButton("+ Sparkles", function() pcall(function() if not par:FindFirstChildOfClass("Sparkles") then Instance.new("Sparkles").Parent=par end end) end)
                        end
                        RPButton("Reset Particles", function()
                                pcall(function()
                                        obj.Rate=20; obj.Lifetime=NumberRange.new(1,5)
                                        obj.Speed=NumberRange.new(10,10); obj.Drag=0
                                        obj.LightEmission=0; obj.Color=ColorSequence.new(Color3.new(1,1,1))
                                end)
                        end)
                end

                if obj:IsA("Fire") or obj:IsA("Smoke") or obj:IsA("Sparkles") then
                        RPSection("Particle FX  ["..cname.."]")
                        RPToggle("Enabled", obj.Enabled, function(on) pcall(function() obj.Enabled=on end) end)
                        if obj:IsA("Fire") then
                                RPColorPicker("Fire Color", obj.Color, function(c) pcall(function() obj.Color=c end) end)
                                RPColorPicker("Secondary Color", obj.SecondaryColor, function(c) pcall(function() obj.SecondaryColor=c end) end)
                                RPSlider("Size",0,30,obj.Size,function(vv) pcall(function() obj.Size=vv end) end)
                                RPSlider("Heat",0,30,obj.Heat,function(vv) pcall(function() obj.Heat=vv end) end)
                        elseif obj:IsA("Smoke") then
                                RPColorPicker("Smoke Color", obj.Color, function(c) pcall(function() obj.Color=c end) end)
                                RPSlider("Size",0,10,obj.Size,function(vv) pcall(function() obj.Size=vv end) end)
                                RPSlider("Rise Velocity",0,30,obj.RiseVelocity,function(vv) pcall(function() obj.RiseVelocity=vv end) end)
                                RPSlider("Opacity",0,1,obj.Opacity,function(vv) pcall(function() obj.Opacity=vv end) end)
                        elseif obj:IsA("Sparkles") then
                                RPColorPicker("Sparkle Color", obj.SparkleColor, function(c) pcall(function() obj.SparkleColor=c end) end)
                        end
                end

                -- ==============  LIGHTS  ==============
                if obj:IsA("Light") then
                        RPSection("Lights  ["..cname.."]")
                        RPToggle("Enabled", obj.Enabled, function(on)
                                pcall(function() obj.Enabled=on end); RecordOverride(obj,"Enabled",on)
                        end)
                        RPColorPicker("Light Color", obj.Color, function(c)
                                pcall(function() obj.Color=c end); RecordOverride(obj,"Color",c)
                        end)
                        RPToggle("RGB Light", false, function(on)
                                local key="l_"..tostring(obj)
                                if _rpRgbFx[key] then _rpRgbFx[key]=false end
                                if on then
                                        _rpRgbFx[key]=true
                                        task.spawn(function()
                                                while _rpRgbFx[key] do
                                                        pcall(function() obj.Color=Color3.fromHSV((tick()%3)/3,1,1) end)
                                                        task.wait(0.05)
                                                end
                                        end)
                                end
                        end)
                        RPSlider("Brightness",0,100,obj.Brightness,function(vv)
                                pcall(function() obj.Brightness=vv end); RecordOverride(obj,"Brightness",vv)
                        end)
                        RPSlider("Range",0,200,obj.Range,function(vv)
                                pcall(function() obj.Range=vv end); RecordOverride(obj,"Range",vv)
                        end)
                        if obj:IsA("SpotLight") then
                                RPSlider("Angle",0,180,obj.Angle,function(vv) pcall(function() obj.Angle=vv end) end)
                        end
                        RPToggle("Shadows", obj.Shadows, function(on)
                                pcall(function() obj.Shadows=on end); RecordOverride(obj,"Shadows",on)
                        end)
                        RPToggle("Flicker", false, function(on)
                                local key="flk_"..tostring(obj)
                                if _rpLightFx[key] then _rpLightFx[key]=false end
                                if on then
                                        local base=obj.Brightness
                                        _rpLightFx[key]=true
                                        task.spawn(function()
                                                while _rpLightFx[key] do
                                                        pcall(function() obj.Brightness=base*(0.7+math.random()*0.6) end)
                                                        task.wait(0.05+math.random()*0.1)
                                                end
                                        end)
                                end
                        end)
                        RPToggle("Pulse", false, function(on)
                                local key="pls_"..tostring(obj)
                                if _rpLightFx[key] then _rpLightFx[key]=false end
                                if on then
                                        local base=obj.Brightness
                                        _rpLightFx[key]=true
                                        task.spawn(function()
                                                while _rpLightFx[key] do
                                                        pcall(function() obj.Brightness=base*(0.5+0.5*math.sin(tick()*3)) end)
                                                        task.wait(0.05)
                                                end
                                        end)
                                end
                        end)
                end

                -- Add light buttons when a BasePart is selected
                if obj:IsA("BasePart") then
                        RPSection("Add Lights")
                        local function addLight(cls)
                                if not obj:FindFirstChildOfClass(cls) then Instance.new(cls).Parent=obj end
                                task.defer(function() BuildProperties(obj:FindFirstChildOfClass(cls)) end)
                        end
                        RPButton("+ PointLight",   function() pcall(function() addLight("PointLight") end) end)
                        RPButton("+ SpotLight",    function() pcall(function() addLight("SpotLight") end) end)
                        RPButton("+ SurfaceLight", function() pcall(function() addLight("SurfaceLight") end) end)
                end

                -- ==============  EFFECTS  ==============
                if obj:IsA("BasePart") then
                        RPSection("Effects")
                        if not _rpEffects[obj] then _rpEffects[obj]={} end
                        local efx = _rpEffects[obj]

                        local function toggleEffect(ename, createFn, cleanFn)
                                RPToggle(ename, efx[ename]~=nil, function(on)
                                        if on then
                                                if efx[ename] then pcall(function() efx[ename]:Destroy() end) end
                                                efx[ename] = createFn()
                                        else
                                                if efx[ename] then pcall(function() efx[ename]:Destroy() end); efx[ename]=nil end
                                                if cleanFn then cleanFn() end
                                        end
                                end)
                        end

                        toggleEffect("Glow", function()
                                local hl=Instance.new("Highlight"); hl.Parent=obj
                                hl.FillColor=Color3.fromRGB(255,255,100); hl.OutlineColor=Color3.fromRGB(255,255,0)
                                hl.FillTransparency=0.6; hl.OutlineTransparency=0; return hl
                        end)
                        toggleEffect("Aura", function()
                                local pe=Instance.new("ParticleEmitter"); pe.Parent=obj
                                pe.LightEmission=1; pe.Rate=30
                                pe.Color=ColorSequence.new(Color3.fromRGB(180,100,255))
                                pe.Size=NumberSequence.new({NumberSequenceKeypoint.new(0,0.5),NumberSequenceKeypoint.new(1,0)})
                                pe.Lifetime=NumberRange.new(0.5,1.5); pe.Speed=NumberRange.new(2,5)
                                pe.SpreadAngle=Vector2.new(180,180); return pe
                        end)
                        toggleEffect("Highlight", function()
                                local hl=Instance.new("Highlight"); hl.Parent=obj
                                hl.FillColor=Color3.new(1,1,1); hl.FillTransparency=0.8
                                hl.OutlineColor=Color3.new(1,1,1); return hl
                        end)
                        toggleEffect("Outline", function()
                                local sb=Instance.new("SelectionBox"); sb.Parent=obj; sb.Adornee=obj
                                sb.Color3=Color3.new(1,1,1); sb.LineThickness=0.04; sb.SurfaceTransparency=1; return sb
                        end)
                        toggleEffect("Swing Trails", function()
                                local f=Instance.new("Folder"); f.Name="_teSwingTrail"; f.Parent=obj
                                local a0=Instance.new("Attachment"); a0.CFrame=CFrame.new(0,obj.Size.Y/2,0); a0.Parent=f
                                local a1=Instance.new("Attachment"); a1.CFrame=CFrame.new(0,-obj.Size.Y/2,0); a1.Parent=f
                                local tr=Instance.new("Trail"); tr.Attachment0=a0; tr.Attachment1=a1
                                tr.Lifetime=0.3; tr.MinLength=0; tr.Parent=f; return f
                        end)
                        toggleEffect("Energy Pulse", function()
                                local f=Instance.new("Folder"); f.Name="_teEnergyPulse"; f.Parent=obj
                                local a0=Instance.new("Attachment"); a0.CFrame=CFrame.new(-obj.Size.X/2,0,0); a0.Parent=f
                                local a1=Instance.new("Attachment"); a1.CFrame=CFrame.new(obj.Size.X/2,0,0); a1.Parent=f
                                local bm=Instance.new("Beam"); bm.Attachment0=a0; bm.Attachment1=a1
                                bm.FaceCamera=true; bm.Width0=0.5; bm.Width1=0.5; bm.LightEmission=1
                                bm.Color=ColorSequence.new(Color3.fromRGB(0,200,255)); bm.Parent=f; return f
                        end)
                        toggleEffect("Electric", function()
                                local pe=Instance.new("ParticleEmitter"); pe.Parent=obj
                                pe.LightEmission=0.8; pe.Rate=80
                                pe.Color=ColorSequence.new(Color3.fromRGB(100,200,255))
                                pe.Size=NumberSequence.new({NumberSequenceKeypoint.new(0,0.3),NumberSequenceKeypoint.new(0.5,0.1),NumberSequenceKeypoint.new(1,0)})
                                pe.Lifetime=NumberRange.new(0.1,0.3); pe.Speed=NumberRange.new(5,15)
                                pe.SpreadAngle=Vector2.new(180,180); pe.RotSpeed=NumberRange.new(-500,500); return pe
                        end)
                        toggleEffect("Fire Aura", function()
                                local fire=Instance.new("Fire"); fire.Parent=obj
                                fire.Size=math.clamp(obj.Size.Magnitude*0.5,1,30)
                                fire.Color=Color3.fromRGB(255,100,0); fire.SecondaryColor=Color3.fromRGB(255,200,0); return fire
                        end)
                        toggleEffect("Ice Aura", function()
                                local pe=Instance.new("ParticleEmitter"); pe.Parent=obj
                                pe.LightEmission=0.5; pe.Rate=40
                                pe.Color=ColorSequence.new(Color3.fromRGB(150,220,255))
                                pe.Size=NumberSequence.new({NumberSequenceKeypoint.new(0,0.4),NumberSequenceKeypoint.new(1,0)})
                                pe.Lifetime=NumberRange.new(1,2); pe.Speed=NumberRange.new(1,3)
                                pe.SpreadAngle=Vector2.new(180,180); pe.Rotation=NumberRange.new(-180,180); return pe
                        end)
                        toggleEffect("Galaxy", function()
                                local sp=Instance.new("Sparkles"); sp.Parent=obj
                                sp.SparkleColor=Color3.fromRGB(100,0,255); sp.Enabled=true; return sp
                        end)
                        toggleEffect("Glitch", function()
                                local f=Instance.new("Folder"); f.Name="_teGlitch"; f.Parent=obj
                                local conn=run_service.Heartbeat:Connect(function()
                                        pcall(function()
                                                if obj and obj.Parent and f.Parent then
                                                        obj.Color=Color3.fromHSV(math.fmod(tick()*5,1),1,1)
                                                        obj.Transparency=math.abs(math.sin(tick()*10))*0.5
                                                end
                                        end)
                                end)
                                efx["_glitchConn"]=conn; return f
                        end, function()
                                if efx["_glitchConn"] then efx["_glitchConn"]:Disconnect(); efx["_glitchConn"]=nil end
                                pcall(function() obj.Transparency=0 end)
                        end)
                        toggleEffect("Hologram", function()
                                local hl=Instance.new("Highlight"); hl.Parent=obj
                                hl.FillColor=Color3.fromRGB(0,200,255); hl.OutlineColor=Color3.fromRGB(0,150,255); hl.FillTransparency=0.5
                                pcall(function() obj.Material=Enum.Material.Neon; obj.Color=Color3.fromRGB(0,200,255); obj.Transparency=0.3 end)
                                return hl
                        end)
                        toggleEffect("Crystal", function()
                                pcall(function() obj.Material=Enum.Material.Glass; obj.Transparency=0.3; obj.Reflectance=0.8 end)
                                local hl=Instance.new("Highlight"); hl.Parent=obj
                                hl.FillColor=Color3.fromRGB(200,230,255); hl.FillTransparency=0.7
                                hl.OutlineColor=Color3.fromRGB(150,200,255); return hl
                        end)
                        toggleEffect("Toxic", function()
                                local pe=Instance.new("ParticleEmitter"); pe.Parent=obj
                                pe.LightEmission=0.3; pe.Rate=60
                                pe.Color=ColorSequence.new(Color3.fromRGB(50,255,50))
                                pe.Size=NumberSequence.new({NumberSequenceKeypoint.new(0,0.5),NumberSequenceKeypoint.new(1,0)})
                                pe.Lifetime=NumberRange.new(1,3); pe.Speed=NumberRange.new(2,8)
                                pe.SpreadAngle=Vector2.new(60,60); return pe
                        end)
                        toggleEffect("Void", function()
                                local pe=Instance.new("ParticleEmitter"); pe.Parent=obj
                                pe.LightEmission=0; pe.Rate=40
                                pe.Color=ColorSequence.new({ColorSequenceKeypoint.new(0,Color3.fromRGB(30,0,60)),ColorSequenceKeypoint.new(1,Color3.fromRGB(0,0,0))})
                                pe.Size=NumberSequence.new({NumberSequenceKeypoint.new(0,0.8),NumberSequenceKeypoint.new(1,0)})
                                pe.Lifetime=NumberRange.new(0.5,2); pe.Speed=NumberRange.new(0,3)
                                pe.SpreadAngle=Vector2.new(180,180); pe.LightInfluence=0; return pe
                        end)
                end

                -- ==============  BEAMS / TRAILS  ==============
                if obj:IsA("Beam") or obj:IsA("Trail") then
                        RPSection("Beam / Trail  ["..cname.."]")
                        RPTextBox("Texture ID", obj.Texture, function(t)
                                pcall(function() obj.Texture=t end); RecordOverride(obj,"Texture",t)
                        end)
                        local startC2=Color3.new(1,1,1)
                        pcall(function() startC2=obj.Color.Keypoints[1].Value end)
                        RPColorPicker("Color", startC2, function(c)
                                pcall(function() obj.Color=ColorSequence.new(c) end)
                                RecordOverride(obj,"Color",ColorSequence.new(c))
                        end)
                        RPSlider("Transparency",0,1,obj.Transparency.Keypoints[1].Value,function(vv)
                                pcall(function() obj.Transparency=NumberSequence.new(vv) end)
                        end)
                        RPSlider("Texture Speed",-10,10,obj.TextureSpeed,function(vv)
                                pcall(function() obj.TextureSpeed=vv end)
                        end)
                        if obj:IsA("Beam") then
                                RPSlider("Width0",0,10,obj.Width0,function(vv) pcall(function() obj.Width0=vv end) end)
                                RPSlider("Width1",0,10,obj.Width1,function(vv) pcall(function() obj.Width1=vv end) end)
                                RPSlider("CurveSize0",-20,20,obj.CurveSize0,function(vv) pcall(function() obj.CurveSize0=vv end) end)
                                RPSlider("CurveSize1",-20,20,obj.CurveSize1,function(vv) pcall(function() obj.CurveSize1=vv end) end)
                                RPSlider("Segments",1,100,obj.Segments,function(vv) pcall(function() obj.Segments=vv end) end)
                        end
                end

                -- ==============  EXPLORER UI  ==============
                RPSection("Explorer UI")
                library._copyBuffer = library._copyBuffer or {}

                RPButton("Copy Properties", function()
                        library._copyBuffer = {}
                        for _, p in ipairs({"Color","Material","Transparency","Reflectance","Size","CastShadow","CanCollide","Anchored"}) do
                                local ok2,val=pcall(function() return obj[p] end)
                                if ok2 then library._copyBuffer[p]=val end
                        end
                        RPLbl("Copied!", themes.preset.accent, 12)
                end)
                RPButton("Paste Properties", function()
                        for prop,val in pairs(library._copyBuffer) do pcall(function() obj[prop]=val end) end
                end)
                RPButton("Reset All Overrides", function()
                        library._toolExplorerOverrides = {}
                        RPLbl("Overrides cleared.", themes.preset.unselected_text, 12)
                end, Color3.fromRGB(220,150,80))
        end

        -- Explorer tree
        local function BuildTree()
                for _, c in pairs(LeftPanel:GetChildren()) do
                        if not c:IsA("UIListLayout") and not c:IsA("UIPadding") then c:Destroy() end
                end
                _selectedRow = nil
                ClearRP()
                RPLbl("Select a part to edit properties.", themes.preset.unselected_text)

                local function AddHeader(text)
                        local hRow = library:create("Frame", {
                                Size = UDim2.new(1, 0, 0, 24),
                                BackgroundColor3 = Color3.fromRGB(20, 20, 20),
                                BorderSizePixel = 0,
                                ZIndex = 104,
                                Parent = LeftPanel,
                        })
                        library:create("UIStroke", {
                                Parent = hRow,
                                Color = Color3.fromRGB(30, 30, 30),
                                Thickness = 1,
                                LineJoinMode = Enum.LineJoinMode.Miter,
                        })
                        local bottomLine = library:create("Frame", {
                                Size = UDim2.new(1, 0, 0, 1),
                                Position = UDim2.new(0, 0, 1, -1),
                                BackgroundColor3 = themes.preset.accent,
                                BorderSizePixel = 0,
                                ZIndex = 105,
                                Parent = hRow,
                        })
                        library:apply_theme(bottomLine, "accent", "BackgroundColor3")
                        local lbl = library:create("TextLabel", {
                                Text = text:upper(),
                                Size = UDim2.new(1, -8, 1, 0),
                                Position = UDim2.new(0, 8, 0, 0),
                                BackgroundTransparency = 1,
                                FontFace = library.font,
                                TextSize = 10,
                                TextColor3 = themes.preset.accent,
                                TextXAlignment = Enum.TextXAlignment.Left,
                                ZIndex = 105,
                                Parent = hRow,
                                TextStrokeTransparency = 0,
                                TextStrokeColor3 = Color3.fromRGB(0, 0, 0),
                        })
                        library:apply_theme(lbl, "accent", "TextColor3")
                end

                local function AddNode(label, obj)
                        local row = library:create("Frame", {
                                Size = UDim2.new(1, 0, 0, 18),
                                BackgroundTransparency = 1,
                                BorderSizePixel = 0,
                                ZIndex = 104,
                                Parent = LeftPanel,
                        })
                        library:create("UIPadding", { PaddingLeft = UDim.new(0, 12), Parent = row })
                        
                        local selectBar = library:create("Frame", {
                                Size = UDim2.new(0, 2, 1, 0),
                                Position = UDim2.new(0, -12, 0, 0),
                                BackgroundColor3 = themes.preset.accent,
                                BorderSizePixel = 0,
                                Visible = false,
                                ZIndex = 105,
                                Parent = row,
                        })
                        library:apply_theme(selectBar, "accent", "BackgroundColor3")

                        local lbl = library:create("TextLabel", {
                                Text = label,
                                Size = UDim2.new(1, 0, 1, 0),
                                BackgroundTransparency = 1,
                                FontFace = library.font,
                                TextSize = 11,
                                TextColor3 = themes.preset.unselected_text,
                                TextXAlignment = Enum.TextXAlignment.Left,
                                ZIndex = 105,
                                Parent = row,
                                TextStrokeTransparency = 0,
                                TextStrokeColor3 = Color3.fromRGB(0, 0, 0),
                        })
                        if obj then
                                local btn = library:create("TextButton", {
                                        Size = UDim2.new(1, 0, 1, 0),
                                        BackgroundTransparency = 1,
                                        Text = "",
                                        ZIndex = 106,
                                        Parent = row,
                                })
                                btn.MouseEnter:Connect(function()
                                        if row ~= _selectedRow then
                                                row.BackgroundTransparency = 0.85
                                                row.BackgroundColor3 = themes.preset.accent
                                        end
                                end)
                                btn.MouseLeave:Connect(function()
                                        if row ~= _selectedRow then row.BackgroundTransparency = 1 end
                                end)
                                btn.MouseButton1Click:Connect(function()
                                        if _selectedRow and _selectedRow ~= row then
                                                _selectedRow.BackgroundTransparency = 1
                                                local oldBar = _selectedRow:FindFirstChild("Frame")
                                                if oldBar then oldBar.Visible = false end
                                                local oldLbl = _selectedRow:FindFirstChildOfClass("TextLabel")
                                                if oldLbl then oldLbl.TextColor3 = themes.preset.unselected_text end
                                        end
                                        _selectedRow = row
                                        row.BackgroundTransparency = 0.92
                                        row.BackgroundColor3 = themes.preset.accent
                                        selectBar.Visible = true
                                        lbl.TextColor3 = Color3.fromRGB(255, 255, 255)
                                        BuildProperties(obj)
                                        library._expSelected = obj
                                end)
                        end
                        return row
                end

                local function AddFolder(label, items)
                        local expanded = true
                        local folderRows = {}
                        local folderRow = library:create("Frame", {
                                Size = UDim2.new(1, 0, 0, 18),
                                BackgroundColor3 = Color3.fromRGB(26, 26, 26),
                                BorderSizePixel = 0,
                                ZIndex = 104,
                                Parent = LeftPanel,
                        })
                        library:create("UIStroke", {
                                Parent = folderRow,
                                Color = Color3.fromRGB(19, 19, 19),
                                Thickness = 1,
                                LineJoinMode = Enum.LineJoinMode.Miter,
                        })
                        local arrowLbl = library:create("TextLabel", {
                                Text = "v",
                                Size = UDim2.new(0, 12, 1, 0),
                                Position = UDim2.new(1, -16, 0, 0),
                                BackgroundTransparency = 1,
                                FontFace = library.font,
                                TextSize = 10,
                                TextColor3 = themes.preset.unselected_text,
                                TextXAlignment = Enum.TextXAlignment.Center,
                                ZIndex = 105,
                                Parent = folderRow,
                        })
                        library:create("TextLabel", {
                                Text = label,
                                Size = UDim2.new(1, -24, 1, 0),
                                Position = UDim2.new(0, 8, 0, 0),
                                BackgroundTransparency = 1,
                                FontFace = library.font,
                                TextSize = 11,
                                TextColor3 = Color3.fromRGB(170, 170, 170),
                                TextXAlignment = Enum.TextXAlignment.Left,
                                ZIndex = 105,
                                Parent = folderRow,
                                TextStrokeTransparency = 0,
                                TextStrokeColor3 = Color3.fromRGB(0, 0, 0),
                        })
                        for idx, obj in ipairs(items) do
                                local r = AddNode("  " .. obj.Name .. " [" .. obj.ClassName .. "]", obj)
                                if r then
                                        library:create("UIPadding", { PaddingLeft = UDim.new(0, 10), Parent = r })
                                        table.insert(folderRows, r)
                                end
                        end
                        local fBtn = library:create("TextButton", {
                                Size = UDim2.new(1, 0, 1, 0),
                                BackgroundTransparency = 1,
                                Text = "",
                                ZIndex = 106,
                                Parent = folderRow,
                        })
                        fBtn.MouseButton1Click:Connect(function()
                                expanded = not expanded
                                arrowLbl.Text = expanded and "v" or ">"
                                for _, r in ipairs(folderRows) do r.Visible = expanded end
                        end)
                end

                local function ScanTools(container, tag)
                        local tools = {}
                        for _, v in pairs(container:GetChildren()) do
                                if v:IsA("Tool") then table.insert(tools, v) end
                        end
                        for _, tool in ipairs(tools) do
                                AddHeader(tag .. " " .. tool.Name)
                                local group1 = {}
                                for _, desc in ipairs(tool:GetDescendants()) do
                                        local g1 = desc:IsA("BasePart") or desc:IsA("SpecialMesh") or desc:IsA("Decal") or desc:IsA("Texture") or desc:IsA("PointLight") or desc:IsA("SpotLight") or desc:IsA("SurfaceLight") or desc:IsA("ParticleEmitter") or desc:IsA("Beam") or desc:IsA("Trail")
                                        if g1 then table.insert(group1, desc) end
                                end
                                if #group1 > 0 then AddFolder("Visuals / Effects", group1) end
                                if #group1 == 0 then AddNode("  (no supported objects)", nil) end
                        end
                        return #tools
                end

                local n1 = ScanTools(lp.Character or Instance.new("Folder"), "[Eq]  ")
                local n2 = ScanTools(lp.Backpack or Instance.new("Folder"), "[Bp]  ")

                if n1 == 0 and n2 == 0 then
                        RPLbl("No tools found. Click refresh.", themes.preset.unselected_text)
                end
        end

        RefreshBtn.MouseButton1Click:Connect(function()
                BuildTree()
        end)

        BuildTree()
end

function library:panel(properties)
        if library.__panel == true then
                return
        end

        library.__panel = true

        local cfg = {
                name = properties.name or "Are you sure?",
                options = properties.options or { "Confirm", "Discard" },
                callback = properties.callback or function() end,
        }

        local panel_main_frame = library:create("Frame", {
                Parent = library.gui,
                Name = "",
                BackgroundTransparency = 0.4000000059604645,
                Size = UDim2.new(1, 0, 1, 0),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                ZIndex = 3,
                BorderSizePixel = 0,
                BackgroundColor3 = Color3.fromRGB(0, 0, 0),
        })

        local holder = library:create("Frame", {
                Parent = panel_main_frame,
                Name = "",
                BorderColor3 = Color3.fromRGB(19, 19, 19),
                AnchorPoint = Vector2.new(0.5, 0.5),
                BackgroundTransparency = 1,
                Position = UDim2.new(0.5, 0, 0.5, 0),
                ZIndex = 4,
                AutomaticSize = Enum.AutomaticSize.XY,
                BackgroundColor3 = Color3.fromRGB(40, 40, 40),
        })

        local inline1 = library:create("Frame", {
                Parent = holder,
                Name = "",
                BorderColor3 = Color3.fromRGB(8, 8, 8),
                AutomaticSize = Enum.AutomaticSize.XY,
                BackgroundColor3 = Color3.fromRGB(56, 56, 56),
        })

        local main = library:create("Frame", {
                Parent = inline1,
                Name = "",
                Position = UDim2.new(0, 4, 0, 4),
                BorderColor3 = Color3.fromRGB(26, 26, 26),
                Size = UDim2.new(1, -8, 1, -8),
                BorderSizePixel = 2,
                BackgroundColor3 = Color3.fromRGB(26, 26, 26),
        })

        local UIStroke = library:create("UIStroke", {
                Parent = main,
                Name = "",
                Color = Color3.fromRGB(57, 57, 57),
                LineJoinMode = Enum.LineJoinMode.Miter,
        })

        local tabs = library:create("Frame", {
                Parent = main,
                Name = "",
                Position = UDim2.new(0, 8, 0, 8),
                BorderColor3 = Color3.fromRGB(8, 8, 8),
                Size = UDim2.new(1, -16, 1, -16),
                BorderSizePixel = 2,
                BackgroundColor3 = themes.preset.bg,
        })
        library:apply_theme(tabs, "bg", "BackgroundColor3")

        local UIStroke = library:create("UIStroke", {
                Parent = tabs,
                Name = "",
                Color = Color3.fromRGB(57, 57, 57),
                LineJoinMode = Enum.LineJoinMode.Miter,
        })

        local UIPadding = library:create("UIPadding", {
                Parent = tabs,
                Name = "",
                PaddingTop = UDim.new(0, 5),
                PaddingBottom = UDim.new(0, 22),
                PaddingRight = UDim.new(0, 20),
                PaddingLeft = UDim.new(0, 20),
        })

        local aimbot = library:create("TextLabel", {
                Parent = tabs,
                Name = "",
                FontFace = library.font,
                LineHeight = 1.2000000476837158,
                TextStrokeTransparency = 0.5,
                AnchorPoint = Vector2.new(0.5, 0),
                TextSize = 12,
                Size = UDim2.new(0, 0, 0, 11),
                TextColor3 = Color3.fromRGB(170, 170, 170),
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                Text = cfg.name,
                BackgroundTransparency = 1,
                Position = UDim2.new(0.5, 0, 0, 8),
                BorderSizePixel = 0,
                TextYAlignment = Enum.TextYAlignment.Top,
                AutomaticSize = Enum.AutomaticSize.XY,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local UIPadding = library:create("UIPadding", {
                Parent = aimbot,
                Name = "",
                PaddingTop = UDim.new(0, 6),
        })

        local UIListLayout = library:create("UIListLayout", {
                Parent = tabs,
                Name = "",
                SortOrder = Enum.SortOrder.LayoutOrder,
                HorizontalAlignment = Enum.HorizontalAlignment.Center,
                Padding = UDim.new(0, 4),
        })

        local Frame = library:create("Frame", {
                Parent = tabs,
                Name = "",
                BorderColor3 = Color3.fromRGB(0, 0, 0),
                BorderSizePixel = 0,
                AutomaticSize = Enum.AutomaticSize.Y,
                BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        })

        local UIListLayout = library:create("UIListLayout", {
                Parent = Frame,
                Name = "",
                SortOrder = Enum.SortOrder.LayoutOrder,
                HorizontalAlignment = Enum.HorizontalAlignment.Center,
                Padding = UDim.new(0, 3),
        })

        local UIPadding = library:create("UIPadding", {
                Parent = Frame,
                Name = "",
        })



        for _, v in next, cfg.options do
                local button_inline = library:create("Frame", {
                        Parent = Frame,
                        Name = "",
                        Position = UDim2.new(0, 0, 0, 4),
                        BorderColor3 = Color3.fromRGB(19, 19, 19),
                        Size = UDim2.new(0, 130, 0, 16),
                        BorderSizePixel = 0,
                        BackgroundColor3 = Color3.fromRGB(8, 8, 8),
                })

                local button = library:create("TextButton", {
                        Parent = button_inline,
                        Name = "",
                        FontFace = library.font,
                        TextColor3 = Color3.fromRGB(170, 170, 170),
                        BorderColor3 = Color3.fromRGB(56, 56, 56),
                        Text = v,
                        TextStrokeTransparency = 0.5,
                        Position = UDim2.new(0, 2, 0, 2),
                        Size = UDim2.new(1, -4, 1, -4),
                        TextSize = 12,
                        BackgroundColor3 = Color3.fromRGB(38, 38, 38),
                })

                button.MouseButton1Click:Connect(function()
                        cfg.callback(v)
                        panel_main_frame:Destroy()
                        library.__panel = false
                end)
        end
end

return library