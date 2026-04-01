# Changelog

API changes and updates. For the full development history, see our GitHub.

---

## v1.0 — March 2026

**Production launch.**

- 311 cognitive abilities across 6 reasoning dimensions (Causal, Temporal, Spatial, Simulation, Abstraction, Metacognition)
- Two modes: `single` (1 ability per call) and `multi` (4 synergized abilities per call)
- Response keys: `single_ability` and `multi_ability`
- Canonical response sections: [NEGATIVE GATE], [REASONING TOPOLOGY], [TARGET PATTERN], [COGNITIVE PAYLOAD], [FALSIFICATION TEST]
- Rate limiting: 100 requests/minute per key
- Tier-based monthly quotas: Free (100 total), Ki (10,000/month), Haki (50,000/month)
- Mode gating: multi mode requires Haki plan
- Endpoint: `POST /logicv1/`

### Stability guarantee

The v1 response schema will not change without a new version path. Field names, section delimiters, and JSON structure are frozen for v1.
