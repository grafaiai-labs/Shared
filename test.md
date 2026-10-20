export EDINET_API_KEY="*****"

curl -s -G "https://api.edinet-fsa.go.jp/api/v2/documents.json" \
  --data-urlencode "date=2026-06-22" \
  --data-urlencode "type=2" \
  -H "Ocp-Apim-Subscription-Key: ${EDINET_API_KEY}"
