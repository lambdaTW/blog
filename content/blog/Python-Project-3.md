+++
title = "Python Project (3) Test Your Project"
author = "lambda@lambda.tw"
categories = ["python", "project", "test"]
tags = ["python", "project", "test"]
date = "2021-04-24"
description = "Test your project"
featured = ""
featuredalt = ""
featuredpath = ""
linktitle = ""
type = "post"
+++
# Test
寫測試可以減少改壞以前的東西，加速開發，在原生的 Python 就有提供測試的方法 [`unittest`](https://docs.python.org/3/library/unittest.html)，然而隨著越來越多的套件支援與其脫鉤，如果是開新專案，可以試著從一開始就使用 [`pytest`](https://github.com/pytest-dev/pytest)

## Pytest
該套件支援簡單的測試方法，多個套件支援其測試，例如在寫 `Django Channels` 時異步測試就推薦使用 `pytest` 可以支援其異步測試，該套件也支援 `pyproject.toml` 當作設定檔

### Usage
```shell-script
pip install pytest

pytest tests
=================================================== test session starts ===================================================
platform darwin -- Python 3.8.5, pytest-6.2.3, py-1.10.0, pluggy-0.13.1 -- /Users/super/project/prj/venv/bin/python3
cachedir: .pytest_cache
django: settings: settings (from ini)
rootdir: /Users/super/project/prj, configfile: pyproject.toml
plugins: cov-2.11.1, django-4.1.0
collected 26 items

Creating test database for alias 'default'...
tests/test_admin/test_actions.py::TestCreateTransformationAction::test_transformation_action_check_fk_values PASSED
tests/test_models/test_copy.py::TestInnerCopy::test_copy_to_new_diagram_inner_attr_are_copied PASSED
tests/test_models/test_copy.py::TestInnerCopy::test_copy_to_new_diagram_nodes_will_be_copied PASSED
tests/test_models/test_copy.py::TestInnerCopy::test_copy_to_new_diagram_transitions_will_be_copied PASSED
tests/test_models/test_copy.py::TestDeepCopy::test_copy_to_new_diagram_inner_attr_are_copied PASSED
tests/test_models/test_copy.py::TestDeepCopy::test_copy_to_new_diagram_nodes_will_be_copied PASSED
tests/test_models/test_copy.py::TestDeepCopy::test_copy_to_new_diagram_transitions_will_be_copied PASSED
tests/test_models/test_copy.py::TestDeepCopy::test_deep_inner_copy_to_new_diagram_inner_attr_are_copied PASSED
tests/test_models/test_freeze.py::TestFreeze::test_freeze_diagram_will_create_static_diagram PASSED
tests/test_models/test_freeze.py::TestFreeze::test_freeze_diagram_with_inner_diagram PASSED
tests/test_models/test_freeze.py::TestFreeze::test_freeze_node_will_take_permissions PASSED
tests/test_models/test_freeze.py::TestFreeze::test_freeze_node_will_take_updateable_fields PASSED
tests/test_models/test_freeze.py::TestFreeze::test_freeze_transformation_will_take_all_things PASSED
tests/test_models/test_freeze.py::TestFreeze::test_freeze_uuid_pk_mode PASSED
tests/test_models/test_inner_enter.py::TestDeepInnerEnter::test_deep_enter PASSED
tests/test_models/test_inner_enter.py::TestDeepInnerEnter::test_deep_leave PASSED
tests/test_models/test_inner_enter.py::TestDeepInnerEnter::test_leave_inner_deep_will_not_present PASSED
tests/test_models/test_inner_enter.py::TestInnerEnter::test_enter_inner_forward_node_will_check_forward_number PASSED
tests/test_models/test_inner_enter.py::TestInnerEnter::test_enter_node_with_inner_will_create_init_nodes_present PASSED
tests/test_models/test_inner_enter.py::TestInnerEnter::test_enter_rollback_will_create_outer_rollback_node_present PASSED
tests/test_models/test_nodes.py::TestFreeze::test_enter_node_will_change_present_node PASSED
tests/test_models/test_permissions.py::TestFreeze::test_get_permission_pks PASSED
tests/test_utils/test_context_utils.py::TestIterKeys::test_simple_case PASSED
tests/test_utils/test_parser_utils.py::TestIterKeys::test_deep_comparison PASSED
tests/test_utils/test_parser_utils.py::TestIterKeys::test_has_attribute PASSED
tests/test_utils/test_parser_utils.py::TestIterKeys::test_simple_case PASSED
Destroying test database for alias 'default'...
```

### Configuration
```toml
# pyproject.toml
[tool.pytest.ini_options]
addopts = "--tb=short -p no:warnings -s -v -ra"
DJANGO_SETTINGS_MODULE = "settings"
# -- recommended but optional:
python_files = [
  "tests.py",
  "test_*.py",
  "*_tests.py",
]
```

## Coverage
跑了測試還不夠，為了要知道哪些程式碼有被測試到，甚至是哪些程式碼被哪些測試程式測試，我們可以使用 `coverage` 套件來達成，該套件也支援 `pyproject.toml`

### Normal usage
```shell-script
pip install coverage
python -m unittest discover
```

### Integrate with pytest
```
pip install coverage
coverage run --source=prj -m pytest
coverage report -m
Name                              Stmts   Miss Branch BrPart  Cover   Missing
----------------------------------------------------------------------------------------------------------------------------
prj/__init__.py                       1      0      0      0   100%
prj/admin/__init__.py                28      9      2      0    63%   17-35
prj/admin/actions.py                 64     33     14      1    44%   14->30, 44-58, 61-82, 103-122
prj/admin/base.py                    47     47     14      0     0%   1-95
prj/admin/helpers.py                 18     11      4      0    32%   8, 13, 18-19, 22-32
prj/apps.py                           3      3      0      0     0%   1-5
prj/models.py                       195      7     67      5    95%   24, 45, 65, 121, 129, 239->238, 373->376, 381->390, 422, 458
prj/tests/__init__.py                 0      0      0      0   100%
prj/utils/__init__.py                 0      0      0      0   100%
prj/utils/ast_parser_utils.py        16      0     10      1    96%   27->exit
prj/utils/rule_context_utils.py       7      1      4      2    73%   9
----------------------------------------------------------------------------------------------------------------------------
TOTAL                               379    111    115      9    70%
```

### Configuration
```toml
# pyproject.toml
[tool.coverage.report]
exclude_lines = [
  "pragma: no cover",
  "def __repr__",
  "if __name__ == .__main__.:",
  "nocov",
  "if TYPE_CHECKING:",
]

[tool.coverage.run]
# Activating branch coverage is super important
branch = true
omit = [
  "*/migrations/*"
]
source = "prj"
```

## pytest-cov
為了讓 pytest 執行時不要那麼麻煩改用 `coverage`，我們使用該套件去讓 `pytest` 直接整合 `coverage` 以後寫測試就只要下 `pytest --cov=prj` 就好了

### Usage
```shell-script
pip install pytest-cov
pytest --cov=prj
=================================================== test session starts ===================================================
platform darwin -- Python 3.8.5, pytest-6.2.3, py-1.10.0, pluggy-0.13.1 -- /Users/super/project/prj/venv/bin/python3
cachedir: .pytest_cache
django: settings: settings (from ini)
rootdir: /Users/super/project/prj, configfile: pyproject.toml
plugins: cov-2.11.1, django-4.1.0
collected 26 items


Creating test database for alias 'default'...
tests/test_admin/test_actions.py::TestCreateTransformationAction::test_transformation_action_check_fk_values PASSED
tests/test_models/test_copy.py::TestInnerCopy::test_copy_to_new_diagram_inner_attr_are_copied PASSED
tests/test_models/test_copy.py::TestInnerCopy::test_copy_to_new_diagram_nodes_will_be_copied PASSED
tests/test_models/test_copy.py::TestInnerCopy::test_copy_to_new_diagram_transitions_will_be_copied PASSED
tests/test_models/test_copy.py::TestDeepCopy::test_copy_to_new_diagram_inner_attr_are_copied PASSED
tests/test_models/test_copy.py::TestDeepCopy::test_copy_to_new_diagram_nodes_will_be_copied PASSED
tests/test_models/test_copy.py::TestDeepCopy::test_copy_to_new_diagram_transitions_will_be_copied PASSED
tests/test_models/test_copy.py::TestDeepCopy::test_deep_inner_copy_to_new_diagram_inner_attr_are_copied PASSED
tests/test_models/test_freeze.py::TestFreeze::test_freeze_diagram_will_create_static_diagram PASSED
tests/test_models/test_freeze.py::TestFreeze::test_freeze_diagram_with_inner_diagram PASSED
tests/test_models/test_freeze.py::TestFreeze::test_freeze_node_will_take_permissions PASSED
tests/test_models/test_freeze.py::TestFreeze::test_freeze_node_will_take_updateable_fields PASSED
tests/test_models/test_freeze.py::TestFreeze::test_freeze_transformation_will_take_all_things PASSED
tests/test_models/test_freeze.py::TestFreeze::test_freeze_uuid_pk_mode PASSED
tests/test_models/test_inner_enter.py::TestDeepInnerEnter::test_deep_enter PASSED
tests/test_models/test_inner_enter.py::TestDeepInnerEnter::test_deep_leave PASSED
tests/test_models/test_inner_enter.py::TestDeepInnerEnter::test_leave_inner_deep_will_not_present PASSED
tests/test_models/test_inner_enter.py::TestInnerEnter::test_enter_inner_forward_node_will_check_forward_number PASSED
tests/test_models/test_inner_enter.py::TestInnerEnter::test_enter_node_with_inner_will_create_init_nodes_present PASSED
tests/test_models/test_inner_enter.py::TestInnerEnter::test_enter_rollback_will_create_outer_rollback_node_present PASSED
tests/test_models/test_nodes.py::TestFreeze::test_enter_node_will_change_present_node PASSED
tests/test_models/test_permissions.py::TestFreeze::test_get_permission_pks PASSED
tests/test_utils/test_context_utils.py::TestIterKeys::test_simple_case PASSED
tests/test_utils/test_parser_utils.py::TestIterKeys::test_deep_comparison PASSED
tests/test_utils/test_parser_utils.py::TestIterKeys::test_has_attribute PASSED
tests/test_utils/test_parser_utils.py::TestIterKeys::test_simple_case PASSED
Destroying test database for alias 'default'...


---------- coverage: platform darwin, python 3.8.5-final-0 -----------
Name                              Stmts   Miss Branch BrPart  Cover
----------------------------------------------------------------------
prj/__init__.py                       1      0      0      0   100%
prj/admin/__init__.py                28      9      2      0    63%
prj/admin/actions.py                 64     33     14      1    44%
prj/admin/base.py                    47     47     14      0     0%
prj/admin/helpers.py                 18     11      4      0    32%
prj/apps.py                           3      3      0      0     0%
prj/models.py                       195      7     67      5    95%
prj/tests/__init__.py                 0      0      0      0   100%
prj/utils/__init__.py                 0      0      0      0   100%
prj/utils/ast_parser_utils.py        16      0     10      1    96%
prj/utils/rule_context_utils.py       7      1      4      2    73%
----------------------------------------------------------------------
TOTAL                               379    111    115      9    70%
```

### Configuration
```toml
# pyproject.toml
[tool.pytest.ini_options]
# 多加一個參數即可
addopts = "--tb=short -p no:warnings -s -v -ra --cov=prj"
DJANGO_SETTINGS_MODULE = "settings"
# -- recommended but optional:
python_files = [
  "tests.py",
  "test_*.py",
  "*_tests.py",
]
```
