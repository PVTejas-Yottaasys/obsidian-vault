14192:
- P26M006WS7 - 9523, 9524

14298:
- P26M00ABRD - 20742
- P26M00ABRE
- P26M00ABRG
- P26M00ABRH
- P26M00ABRI
- P26M00ABRJ
- P26M00ABRK
- P26M00ABRL
- P26M00ABRM
- P26M00ABRN
- P26M00ABRO
- P26M00ABRP
- P26M00ABRQ
- P26M00ABRR - 20655
- P26M00ABRS
- P26M00ABRT

14297:
- P26M00AD8K
- P26M00ASWL - 21841
- P26M00ASWR
- P26M00AD95
- P26M00ASWM - 21842

Initial work was done to support M1, C2, MK9 and OM42 frames
The solution was robust enough to 
Updates made so far:
- Fixed some crashes, improved stability
- Line detection updates: false rejects went from 17% to 5% after this change
  Gains came from having a lot of production data to improve our models
- Some UI improvements (Jexin can provide a detailed list. I remember feedback button, self-hosted dashboard)
Updates we will deploy soon:
- Stopper position: NM379 frames exposed a flawed assumption in our stopper reference detection. This will be deployed along with new JSON update
- New JSON: 4-inch M1 frames (not seen during solution development) and GE39 frames are not compatible with the frame info JSON that was finalised during initial deployment. New JSON format is finalised, but 