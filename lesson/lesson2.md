# PocketFlow之**结构化输出**(Structured Output)-提取简历数据

本小节你将了解到如何使用 PocketFlow 来实现结构化输出。


## 项目流程图

```mermaid
flowchart TD
    ResumeParseNode
```


## 前置条件: [环境安装与配置](./init-env.md)

## 步骤1: 创建项目目录结构

```bash
# 进入虚拟环境
cd p3
# 进入项目目录
cd workspace
# 新建目录
mkdir data
# 复制提供的 resume.txt 到你的 data 目录下
[resume.txt](../data/resume.txt)
# 创建项目文件
touch main2.py flows/resumeParseFlow.py
```


## 步骤2: 定义简历解析节点


在 `resumeParseFlow.py` 中，添加以下代码:

```python
from pocketflow import Node,Flow
from utils.call_llm import call_llm
import yaml

class ResumeParseNode(Node):
    # 获取简历数据
    def prep(self, shared):
        return shared['resume_text']
    # 解析简历并按需求的格式输出
    def exec(self, resume_text):
        prompt = f"""
        请从这份简历中提取以下信息，并将其格式化为 YAML：
        {resume_text}
        现在请按照以下格式输出：
        ```yaml
        name: 名字  
        email: xxx@xxx.com  
        experience:  
            - title: 职位名称1  
              company: 公司1
            - title: 职位名称2  
              company: 公司2
        skills:  
            - 技能1  
            - 技能2   
            - 技能3
        ```"""
        res = call_llm(prompt)
        yaml_str = res.split("```yaml")[1].split("```")[0].strip()
        structured_data = yaml.safe_load(yaml_str)
        return structured_data
    # 将结果处理后保存
    def post(self, shared, prep_res, exec_res):
        shared['structured_data'] = exec_res

        print("\n=== 简历结构化摘要数据 ===\n")
        print(yaml.dump(exec_res,allow_unicode=True,sort_keys=False))
        print("========================")

# 新建简历处理节点
resume_node = ResumeParseNode()
# 新建简历流程
resume_flow = Flow(start=resume_node)
```

## 步骤3: 创建主程序

在 `main2.py` 中，添加以下代码:

```python
from flows.resumeParseFlow import resume_flow

def main():
    shared = {}
    # 读取简历文件
    with open('./data/resume.txt', 'r', encoding='utf8') as file:
        resume_text = file.read()
    shared['resume_text'] = resume_text
    resume_flow.run(shared)


if __name__ == '__main__':
    main()
```


## 步骤4: 运行程序

```bash
python main2.py
```


## 运行结果

![运行结果](/images/lesson2/1.png)


## 下一课
[PocketFlow之**工作流**(Workflow)-自动化天气信息处理系统](./lesson3.md)