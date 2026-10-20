# 1. APIキーとヘッダーの設定
$apiKey = "cc63eb0c0c4f4bb5a8094078b9b28092"
$headers = @{
    "Ocp-Apim-Subscription-Key" = $apiKey
}

# 2. 書類一覧を取得し、パーソルHD (E21261) かつ有価証券報告書の docID を抽出
$date = "2026-06-22"
$listUrl = "https://api.edinet-fsa.go.jp/api/v2/documents.json?date=$date&type=2"

$response = Invoke-RestMethod -Uri $listUrl -Method Get -Headers $headers
$targetDoc = $response.results | Where-Object {
    $_.edinetCode -eq "E21261" -and $_.docDescription -like "*有価証券報告書*"
}

$docId = $targetDoc.docID
Write-Host "取得対象 docID: $docId"

# 3. XBRL (ZIP) ファイルの取得 (type=1)
$zipUrl = "https://api.edinet-fsa.go.jp/api/v2/documents/$docId?type=1"
Invoke-WebRequest -Uri $zipUrl -Method Get -Headers $headers -OutFile "persol_20260622_xbrl.zip"

# 4. PDF ファイルの取得 (type=2)
$pdfUrl = "https://api.edinet-fsa.go.jp/api/v2/documents/$docId?type=2"
Invoke-WebRequest -Uri $pdfUrl -Method Get -Headers $headers -OutFile "persol_20260622.pdf"

# 5. ファイル確認
Get-ChildItem persol_20260622* | Select-Object Name, Length, LastWriteTime
