+++
title = "Python Project (4) Test More"
author = "lambda@lambda.tw"
categories = ["python", "project", "test"]
tags = ["python", "project", "test"]
date = "2021-04-25"
description = "Test more with tox"
featured = ""
featuredalt = ""
featuredpath = ""
linktitle = ""
type = "post"
+++
# Python Version Tests
如果專案只是一般服務，可能基本測試就足夠了，但是如果是要寫 Library 給別的專案或是開放給大家使用的話，就要考慮更多的相容性問題，例如： Python 版本，相依套件版本，等等，那一般的測試可能就不足以符合這樣的情境，因此我們需要更多的整合測試

## tox
利用 tox 可以測試多種環境，如下圖：
![Tox flow](https://tox.readthedocs.io/en/latest/_images/tox_flow.png)
他可以支援 `.tox` 和 `pyproject.toml` 但是在 `pyproject.toml` 裡面是用字串寫舊有設定格式，由於他會先把你的專案，用 `setup.py` 建立好，用 pip 裝起來後再去執行測試（指令），所以我們要把測試改寫到 tests ，並且儘量用絕對路徑去 import 我們自己的專案，這樣才不會有問題

### Usage
```
pip install tox
tox -e py38-django31
py38-django31 inst-nodeps: /Users/super/project/prj/.tox/.tmp/package/1/prj-0.1.0.tar.gz
py38-django31 installed: asgiref==3.3.4,attrs==20.3.0,coverage==5.5,Django==3.1.8,django-object-actions==3.0.2,prj @ file:///Users/super/project/prj/.tox/.tmp/package/1/prj-0.1.0.tar.gz,iniconfig==1.1.1,packaging==20.9,pluggy==0.13.1,ply==3.11,py==1.10.0,pyparsing==2.4.7,pytest==6.2.3,pytest-cov==2.11.1,pytest-django==4.2.0,python-dateutil==2.8.1,pytz==2021.1,rule-engine==3.2.0,six==1.15.0,sqlparse==0.4.1,toml==0.10.2
py38-django31 run-test-pre: PYTHONHASHSEED='3217434528'
py38-django31 run-test: commands[0] | coverage erase
py38-django31 run-test: commands[1] | pytest tests
============================= test session starts ==============================
platform darwin -- Python 3.8.5, pytest-6.2.3, py-1.10.0, pluggy-0.13.1 -- /Users/super/project/prj/.tox/py38-django31/bin/python
cachedir: .tox/py38-django31/.pytest_cache
django: settings: settings (from ini)
rootdir: /Users/super/project/prj, configfile: pyproject.toml
plugins: cov-2.11.1, django-4.2.0
collecting ... collected 26 items

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


---------- coverage: platform darwin, python 3.8.5-final-0 -----------
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


============================== 26 passed in 4.63s ==============================
Creating test database for alias 'default'...
Destroying test database for alias 'default'...
___________________________________ summary ____________________________________
  py38-django31: commands succeeded
  congratulations :)

```

### Configuration
```toml
# pyproject.toml
legacy_tox_ini = """
[tox]
isolated_build = True
envlist =
    {py38,py39}-django31,
    {py38,py39}-django32,
[testenv]
deps =
    coverage
    pytest-cov
    pytest-django
    django31: django~=3.1.0
    django32: django~=3.2.0
commands =
    coverage erase
    pytest tests
```
