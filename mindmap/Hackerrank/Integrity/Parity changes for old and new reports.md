## Raw Task Dump
- new report depends on integrity_summary_v2 we need this for all older attempts as well (this is a backfill task)
- integrity_status -> when we released proctor mode, then we also started using no proctor report. non proctor = secure mode off or on with things like photo id on. we need to backfill. we want to backfill for whole of 2025. basically for attempts even before proctor mode was released.
- PDF also needs to use integrity status only. (generate integrity status on the go), check if any code change is required 
- integrity_summary -> released for proctor mode, integrity_summary_v2 -> generate for all 2025
  integrity_summary_text -> used for integration, needs to also be regenerated. (do for 2025 attempts)
- MOSS (code similarity) -> either coding or coding approx, it always runs. you'll see flagging due to this but not visible in the report, now for code similarity should also contribute to integrity issue as medium. need to check, main report, excel, pdf, email report.
- spoofing check for non proctor mode. if there is no face and spoofing, only mark no face in proctoring analysis service lambda. spoofing is not in proctor mode.
- filters like suspicious filter (integrity ones), as soon as you select, elasticsearch will return some data. we might have to add/remove data to elasticsearch. (private, preview, production). 
- IES API for non proctor violations as well. (check how search is working and see if i need to send additional tid or email data from api itself)
- for copy paste violations, the violation array is coming empty, even though the integrity_v2 summary has the violation listed. expected behaviour: should redirect to codeplayer with timestamps. this violation isnt present in plagiarism or ml report, so we cant seek to the place in codeplayer. should return array of timestamps scoped to question ids (since codeplayer only shows oen question at a time) [https://lpfe-parity.private.hackerrank.link/work/tests/430864/candidates?c=all&page=1&openReport=7399846](https://lpfe-parity.private.hackerrank.link/work/tests/430864/candidates?c=all&page=1&openReport=7399846)
for testing have proctor, non proctor (secure mode) reports (old and new) and compare
### Open Questions
- backfill dates?