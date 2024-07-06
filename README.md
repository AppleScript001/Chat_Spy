
--Red = You
--White = Message
--Green = Friend
--Blue = Other Players
--Magenta = Custom Players
--Warnings = Found Messages

FindCommand = "/e Find"

--Put Player ID's Here to Give Them Magenta Color
CustomColors = {
    --Put Player ID's here
}

rconsolename("Private Messages")
rconsoleprint("@@GREEN@@")

local plyr = game.Players.LocalPlayer
local ID = plyr.UserId

local getmsg = game:GetService("ReplicatedStorage"):WaitForChild("DefaultChatSystemChatEvents"):WaitForChild("OnMessageDoneFiltering")

Messages = {}

local LoweredFindCommand = string.lower(FindCommand)

plyr.Chatted:Connect(function(msg, r)
    if string.find(msg, FindCommand) then
        local NewMessage = string.lower(msg)
        local newString = string.split(NewMessage, " ")
        for _, z in pairs(Messages) do
            local v = string.lower(z)
            local New_String = string.split(v, " ")
            for i, x in pairs(New_String) do
                if i >= 4 then
                    if (string.find(New_String[i], newString[3])) and not (string.find(v, LoweredFindCommand)) then
                        rconsolewarn(v)
                    end
                end
            end
        end
    end
end)

function DisplayMessage(msg, player)
    local NewMsg = (msg.."\n")
    local NewName = (player.DisplayName.." - "..player.Name..": ")
    local NewMessageAndName = (NewName..NewMsg)
    table.insert(Messages, NewMessageAndName)
    if player.Name == game.Players.LocalPlayer.Name then
        rconsoleprint("@@RED@@")
    else
        if table.find(CustomColors, player.UserId) then
            rconsoleprint("@@MAGENTA@@")
        elseif player:IsFriendsWith(ID) then
            rconsoleprint("@@GREEN@@")
        else
            rconsoleprint("@@BLUE@@")
        end
    end
    rconsoleprint(NewName)
    rconsoleprint("@@WHITE@@")
    rconsoleprint(NewMsg)
end

function OnChat(p, msg)
    local NewMsg = (msg.."\n")
    local NewName = (p.DisplayName.." - "..p.Name..": ")
    local NewMessageAndName = (NewName..NewMsg)
    table.insert(Messages, NewMessageAndName)

    msg = msg:gsub("[\n\r]",''):gsub("\t",' '):gsub("[ ]+",' ')
    local hidden = true
    local conn = getmsg.OnClientEvent:Connect(function(packet,channel)
        if packet.SpeakerUserId==p.UserId and packet.Message==msg:sub(#msg-#packet.Message+1) and (channel=="All" or (channel=="Team" and public==false and Players[packet.FromSpeaker].Team==plyr.Team)) then
            hidden = false
        end
    end)
    wait(1)
    conn:Disconnect()
    if hidden then
        DisplayMessage(msg, p)
    end
end

function FoundPlayer(player)
    player.Chatted:Connect(function(msg, p)
        OnChat(player, msg)
    end)
end
for _, v in pairs(game.Players:GetChildren()) do
    FoundPlayer(v)
end
game.Players.PlayerAdded:Connect(FoundPlayer)
