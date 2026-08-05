-- ===== デバッグ用 最小構成 =====
print("スクリプト開始") -- これが表示されないならExecutor自体が動いてない

-- Rayfield がロードできるかテスト
local success, rayfield = pcall(function()
    return loadstring(game:HttpGet("https://raw.githubusercontent.com/shlexware/Rayfield/main/source"))()
end)

if not success then
    warn("Rayfield ロード失敗: ", rayfield)
    -- 代替ロード方法
    local alt = pcall(function()
        return loadstring(game:HttpGet("https://pastebin.com/raw/xxx"))() -- 仮のミラー
    end)
    if alt then print("代替ロード成功") else print("両方失敗") end
else
    print("Rayfield ロード成功")
end

-- ReplicatedStorage の中身を確認
print("GrabEvents:", ReplicatedStorage:FindFirstChild("GrabEvents") and "存在" or "なし")
print("MenuToys:", ReplicatedStorage:FindFirstChild("MenuToys") and "存在" or "なし")

-- ここまで実行できたらGUI表示テスト
if rayfield then
    local testWin = rayfield:CreateWindow({Name = "Test"})
    print("GUI作成完了")
end
