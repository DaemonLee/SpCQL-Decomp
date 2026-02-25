# SpCQL-Decomp  
### A Query Decomposition Dataset for Text-to-Cypher Semantic Parsing

**Current Version: v1.0
**Last Updated: 2026-02-25

SpCQL-Decomp is a decomposition-enhanced version of the original **SpCQL** dataset.  
It introduces structured, dependency-aware sub-query annotations to support research on compositional semantic parsing and multi-hop reasoning for Cypher generation.

---

## 1. Relation to Original SpCQL

This dataset is constructed based on:

> **SpCQL: A Semantic Parsing Dataset for Converting Natural Language into Cypher**  
> CIKM 2022

Official SpCQL repository:  
https://github.com/Guoaibo/Text-to-CQL/tree/main

SpCQL-Decomp preserves the original:

- Natural language query  
- Gold Cypher query  
- Gold answer  

And additionally provides:

- Step-wise decomposition annotations  
- Sub-query templates  
- Generated sub-Cypher queries  
- Dependency structures  
- Entity grounding at each step

The Neo4j graph database required for executing Cypher queries is identical to the original SpCQL dataset. Please refer to the official SpCQL repository for download:Baidu Pan link: https://pan.baidu.com/s/1aqMZFMOOpiB1GWUo5-I7xQ?pwd=b6ix

Note: The original Baidu Pan link for the SpCQL Neo4j database may be inaccessible due to platform restrictions. Please refer to the official SpCQL repository for the latest download updates or alternative access methods.

If you use this dataset, please cite both the original SpCQL paper and our work.

---

## 2. Motivation

Although SpCQL provides high-quality text-to-Cypher pairs, many queries require:

- Multi-hop reasoning  
- Intermediate variable grounding  
- Structured execution planning  

However, the original dataset does not explicitly annotate intermediate reasoning steps.

SpCQL-Decomp addresses this limitation by:

- Decomposing complex Cypher queries into executable sub-queries  
- Making reasoning chains explicit  
- Providing dependency-aware supervision signals  
- Enabling step-by-step training and evaluation  

---

## 3. Decomposed Dataset Format

Each instance is structured as follows:

```json
{
  "query": "列举出鲁迅的一个别名可以吗？",
  "entities": [
    "鲁迅"
  ],
  "decomposition": [
    {
      "id": "Q1",
      "subquery": "定位实体：鲁迅",
      "input_template": "match (q:ENTITY {name:'[ENT1]'}) return q",
      "generated_cypher": "match (q:ENTITY {name:'鲁迅'}) return q",
      "depends_on": [],
      "entities": [
        "鲁迅"
      ]
    },
    {
      "id": "Q2",
      "subquery": "查询鲁迅的别名实体，关系：鲁迅 -【别名】-> 别名实体",
      "input_template": "match (q:ENTITY{name:'[ENT1]'})-[:Relationship{name:'别名'}]->(m) return m",
      "generated_cypher": "match (q:ENTITY{name:'鲁迅'})-[:Relationship{name:'别名'}]->(m) return m",
      "depends_on": [
        "Q1"
      ],
      "entities": [
        "鲁迅"
      ]
    },
    {
      "id": "Q3",
      "subquery": "返回Q2结果的名称属性",
      "input_template": "return m.name",
      "generated_cypher": "return m.name",
      "depends_on": [
        "Q2"
      ],
      "entities": []
    },
    {
      "id": "Q4",
      "subquery": "限制返回1条数据",
      "input_template": "limit 1",
      "generated_cypher": "limit 1",
      "depends_on": [
        "Q3"
      ],
      "entities": [],
      "is_final": true
    }
  ],
  "target_text": "match (q:ENTITY{name:'鲁迅'})-[:Relationship{name:'别名'}]->(m) return m.name limit 1"
}
```

---

## 4. Field Description

### query
Original natural language question.

### entities
Entities explicitly mentioned in the natural language query.

### decomposition
Ordered sub-query reasoning chain.

Each step contains:

- **id**: Unique step identifier (e.g., Q1, Q2, ...)  
- **subquery**: Natural language description of the reasoning step  
- **input_template**: Abstract Cypher template with placeholders  
- **generated_cypher**: A syntactically valid Cypher clause fragment (not a standalone executable query). It relies on variables/schemas from prior steps (via depends_on), and all fragments are composed sequentially to form the full executable target_text.
- **depends_on**: List of prerequisite steps  
- **entities**: Entities involved in the current step  
- **is_final** (optional): Whether this step completes the full query

### target_text
Final complete Cypher query equivalent to the original SpCQL gold query.

---

## 5. Design Principles

### 5.1 Execution-Aligned Decomposition

Each sub-query corresponds to a valid Cypher clause fragment, ensuring:

- Structural correctness  
- Executability after composition  
- Alignment with Neo4j semantics  

### 5.2 Dependency Awareness

The `depends_on` field explicitly encodes:

- Variable flow  
- Logical ordering  
- Multi-hop reasoning chains  

This enables modeling:

- Graph-of-thought reasoning  
- Step-wise planning  
- Hierarchical decoding  

### 5.3 Template Grounding

Each step includes:

- An abstract template  
- A fully grounded Cypher clause  

This supports:

- Template-based generation  
- Slot filling modeling  
- Fine-grained supervision  

---

## 6. Dataset Splits

The dataset follows the same split as the original SpCQL:

- train.json: 6,320 queries
- dev.json: 790 queries  
- test.json: 790 queries  

All decomposition annotations strictly align with the original data split.

---

## 7. Research Applications

SpCQL-Decomp supports research in:

- Text-to-Cypher semantic parsing  
- Compositional generalization  
- Multi-hop reasoning  
- Structured LLM planning  
- Program-of-Thought modeling  
- Step-wise execution supervision  

---

## 8. Evaluation Suggestions

Beyond traditional metrics:

- Exact Match (EM)  
- Execution Accuracy (EX)  

We additionally recommend:

- Sub-query EM  
- Dependency prediction accuracy  
- Step-wise structural alignment  
- Decomposition consistency score  

---

## 9. Citation

If you use this dataset, please cite:

### Original SpCQL

```bibtex
@inproceedings{DBLP:conf/cikm/GuoLXT022,
  author       = {Aibo Guo and
                  Xinyi Li and
                  Guanchen Xiao and
                  Zhen Tan and
                  Xiang Zhao},
  title        = {SpCQL: A Semantic Parsing Dataset for Converting Natural Language into Cypher},
  booktitle    = {Proceedings of the 31st ACM International Conference on Information
                  \& Knowledge Management (CIKM)},
  year         = {2022}
}
```

### SpCQL-Decomp

```bibtex
@inproceedings{li2026spcql_decomp,
  author       = {Dameng Li and
                  Qi Zhou and
                  Donghang Duan and
                  Jing Liu and
                  Chong Mu and
                  Xu Zheng},
  title        = {A Decomposition-Based Framework for Chinese Natural Language to Cypher Query Generation},
  booktitle    = {Proceedings of the 8th International Conference on Natural Language Processing (ICNLP)},
  year         = {2026},
  note         = {Accepted for oral presentation}
}
```
---

## 10. License

This dataset follows the same license as the original SpCQL dataset.  
For commercial usage, please contact the original authors.

---

## 11. Acknowledgements

We sincerely thank the authors of SpCQL for releasing the dataset and Neo4j database, which made this extension possible.

---

## 12. Contact

For questions or issues, please open a GitHub Issue or contact: cuit.ldm66@gmail.com
