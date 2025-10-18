
---

# 2 — Explainable Query Visualizer (query_visualizer.py)

**File:** `src/query_visualizer.py`
```python
#!/usr/bin/env python3
"""
Explainable Query Visualizer (prototype)
- Parses a simplified SQL EXPLAIN-like plan (JSON) and prints human-friendly advice.
Run: python3 src/query_visualizer.py
"""
import json, sys
from typing import Dict, Any, List

SAMPLE_PLAN = {
    "Plan": {
        "Node Type": "Aggregate",
        "Plans": [
            {"Node Type": "Seq Scan", "Relation Name": "orders", "Filter": "status = 'open'", "Cost": 1200},
            {"Node Type": "Index Scan", "Relation Name": "users", "Index Name": "users_pkey", "Cost": 10}
        ],
        "Cost": 1210
    }
}

def advice_from_plan(plan: Dict[str, Any]) -> List[str]:
    adv = []
    node = plan.get("Plan", plan)
    def walk(n):
        t = n.get("Node Type","")
        if t == "Seq Scan":
            adv.append(f"Sequential scan on {n.get('Relation Name')} (cost {n.get('Cost')}). Consider adding an index for common filters: {n.get('Filter')}")
        if t == "Index Scan":
            adv.append(f"Index scan on {n.get('Relation Name')} using {n.get('Index Name')}. Good.")
        for p in n.get("Plans", []):
            walk(p)
    walk(node)
    return adv

def pretty(plan):
    for a in advice_from_plan(plan):
        print(" -", a)

def demo():
    print("Plan advice for sample:")
    pretty(SAMPLE_PLAN)

if __name__ == "__main__":
    demo()
