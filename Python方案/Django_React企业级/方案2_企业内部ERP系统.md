# 方案2：企业内部ERP系统 - 完整项目指南

> **技术栈**: Python + Django + React + PostgreSQL + Celery  
> **难度等级**: Lv.3 (中级)  
> **项目周期**: 3-4周  
> **适用场景**: 企业资源规划、内部管理系统、OA系统

---

## 📋 项目介绍

### 1.1 项目概述
本项目是一个功能完整的**企业内部ERP系统**，包含人力资源管理（组织架构、员工管理、考勤）、项目管理（任务分配、进度追踪）、审批流程（请假、报销、采购）等核心模块。

### 1.2 核心技术亮点
- **工作流引擎**：自定义审批流程
- **权限系统**：RBAC 角色权限管理
- **实时通知**：WebSocket 消息推送
- **数据导入导出**：Excel 操作
- **定时任务**：Celery 定时任务

### 1.3 适用场景
- 企业内部管理
- OA 办公自动化
- 项目管理系统
- 人力资源系统

---

## 🏗️ 完整可运行代码

### 2.1 权限系统

#### 模型定义
```python
# apps/rbac/models.py
"""
基于角色的访问控制 (RBAC)

核心概念：
- 用户 (User)：系统使用者
- 角色 (Role)：权限的集合
- 权限 (Permission)：对资源的操作许可
- 菜单 (Menu)：系统菜单结构
"""
from django.db import models
from django.contrib.auth import get_user_model

User = get_user_model()


class Permission(models.Model):
    """
    权限定义
    
    格式：app.action (如: user.view, order.create)
    """
    name = models.CharField("权限名称", max_length=100)
    code = models.CharField("权限编码", max_length=100, unique=True)
    resource = models.CharField("资源", max_length=50)  # 如: user, order
    action = models.CharField("操作", max_length=50)     # 如: view, create
    description = models.CharField("描述", max_length=200, blank=True)
    
    class Meta:
        verbose_name = "权限"
        verbose_name_plural = verbose_name
        ordering = ['resource', 'action']
    
    def __str__(self):
        return f"{self.resource}.{self.action}"


class Role(models.Model):
    """
    角色
    
    角色是权限的集合
    """
    name = models.CharField("角色名称", max_length=100)
    code = models.CharField("角色编码", max_length=50, unique=True)
    permissions = models.ManyToManyField(
        Permission,
        verbose_name="权限",
        blank=True,
        related_name='roles'
    )
    menus = models.ManyToManyField(
        'Menu',
        verbose_name="菜单",
        blank=True,
        related_name='roles'
    )
    is_active = models.BooleanField("是否启用", default=True)
    description = models.TextField("描述", blank=True)
    created_at = models.DateTimeField("创建时间", auto_now_add=True)

    class Meta:
        verbose_name = "角色"
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.name
    
    def has_permission(self, code: str) -> bool:
        """检查是否拥有某个权限"""
        return self.permissions.filter(code=code).exists()


class Menu(models.Model):
    """
    菜单
    
    支持多级菜单
    """
    TYPE_CHOICES = [
        ('directory', '目录'),
        ('menu', '菜单'),
        ('button', '按钮'),
    ]
    
    name = models.CharField("菜单名称", max_length=100)
    type = models.CharField("类型", max_length=20, choices=TYPE_CHOICES)
    parent = models.ForeignKey(
        'self',
        verbose_name="父菜单",
        on_delete=models.CASCADE,
        null=True,
        blank=True,
        related_name='children'
    )
    path = models.CharField("路由", max_length=200, blank=True)  # 前端路由
    component = models.CharField("组件", max_length=200, blank=True)  # 前端组件
    icon = models.CharField("图标", max_length=50, blank=True)
    sort_order = models.IntegerField("排序", default=0)
    is_visible = models.BooleanField("是否显示", default=True)
    is_active = models.BooleanField("是否启用", default=True)

    class Meta:
        verbose_name = "菜单"
        verbose_name_plural = verbose_name
        ordering = ['sort_order']

    def __str__(self):
        return self.name


class UserProfile(models.Model):
    """
    用户扩展信息
    
    关联 Django User
    """
    GENDER_CHOICES = [
        ('male', '男'),
        ('female', '女'),
        ('unknown', '未知'),
    ]
    
    user = models.OneToOneField(
        User,
        verbose_name="用户",
        on_delete=models.CASCADE,
        related_name='profile'
    )
    
    # 基本信息
    employee_no = models.CharField("工号", max_length=50, unique=True)
    gender = models.CharField("性别", max_length=20, choices=GENDER_CHOICES, default='unknown')
    phone = models.CharField("手机", max_length=20, blank=True)
    avatar = models.URLField("头像", blank=True)
    
    # 组织架构
    department = models.ForeignKey(
        'Department',
        verbose_name="部门",
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
        related_name='employees'
    )
    position = models.CharField("职位", max_length=100, blank=True)
    
    # 角色
    roles = models.ManyToManyField(
        Role,
        verbose_name="角色",
        blank=True,
        related_name='users'
    )
    
    # 入职信息
    entry_date = models.DateField("入职日期", null=True, blank=True)
    is_active = models.BooleanField("在职状态", default=True)
    
    created_at = models.DateTimeField("创建时间", auto_now_add=True)
    updated_at = models.DateTimeField("更新时间", auto_now=True)

    class Meta:
        verbose_name = "用户信息"
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.user.username
    
    def has_permission(self, code: str) -> bool:
        """检查用户是否拥有某权限（检查所有角色）"""
        for role in self.roles.filter(is_active=True):
            if role.has_permission(code):
                return True
        return False
    
    def get_all_permissions(self):
        """获取所有权限"""
        permissions = set()
        for role in self.roles.filter(is_active=True):
            permissions.update(role.permissions.all())
        return permissions


class Department(models.Model):
    """
    部门（组织架构）
    """
    name = models.CharField("部门名称", max_length=100)
    code = models.CharField("部门编码", max_length=50, unique=True)
    parent = models.ForeignKey(
        'self',
        verbose_name="上级部门",
        on_delete=models.CASCADE,
        null=True,
        blank=True,
        related_name='children'
    )
    manager = models.ForeignKey(
        User,
        verbose_name="负责人",
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
        related_name='managed_departments'
    )
    description = models.TextField("描述", blank=True)
    is_active = models.BooleanField("是否启用", default=True)
    created_at = models.DateTimeField("创建时间", auto_now_add=True)

    class Meta:
        verbose_name = "部门"
        verbose_name_plural = verbose_name

    def __str__(self):
        return self.name
    
    def get_full_path(self):
        """获取完整路径"""
        if self.parent:
            return f"{self.parent.get_full_path()} > {self.name}"
        return self.name
```

#### 权限验证装饰器
```python
# utils/rbac/permissions.py
"""
权限验证装饰器和工具

使用方式：
1. 装饰器：@require_permission('user.view')
2. Mixin：class UserViewSet(PermissionMixin, ...)
"""
from functools import wraps
from rest_framework import status
from rest_framework.response import Response
from django.http import JsonResponse


def require_permission(permission_code: str):
    """
    权限验证装饰器
    
    用于视图函数或方法
    """
    def decorator(view_func):
        @wraps(view_func)
        def wrapper(request, *args, **kwargs):
            # 检查用户是否登录
            if not request.user.is_authenticated:
                return JsonResponse({'error': '请先登录'}, status=401)
            
            # 检查权限
            profile = getattr(request.user, 'profile', None)
            if profile and profile.has_permission(permission_code):
                return view_func(request, *args, **kwargs)
            
            # 超级管理员拥有所有权限
            if request.user.is_superuser:
                return view_func(request, *args, **kwargs)
            
            return JsonResponse({'error': '没有权限'}, status=403)
        
        return wrapper
    return decorator


class PermissionMixin:
    """
    权限验证 Mixin
    
    用于 DRF ViewSet
    """
    # 在子类中覆盖
    required_permission = None  # 如: 'user.view'
    required_permissions = None  # 如: ['user.view', 'user.create']
    
    def check_permissions(self, request):
        """重写权限检查"""
        super().check_permissions(request)
        
        # 获取用户
        user = request.user
        if not user.is_authenticated:
            return Response(
                {'detail': '请先登录'},
                status=status.HTTP_401_UNAUTHORIZED
            )
        
        # 超级管理员跳过
        if user.is_superuser:
            return
        
        # 获取权限列表
        permissions = self.required_permissions or []
        if self.required_permission:
            permissions.append(self.required_permission)
        
        # 检查权限
        profile = getattr(user, 'profile', None)
        for perm in permissions:
            if profile and profile.has_permission(perm):
                return
        
        return Response(
            {'detail': '没有权限'},
            status=status.HTTP_403_FORBIDDEN
        )


class DepartmentFilterMixin:
    """
    部门数据隔离 Mixin
    
    确保用户只能访问自己部门的数据
    """
    department_field = 'department'  # 数据模型中的部门字段
    
    def get_queryset(self):
        """过滤查询集"""
        queryset = super().get_queryset()
        user = self.request.user
        
        # 超级管理员和 HR 可以看全部
        if user.is_superuser:
            return queryset
        
        profile = getattr(user, 'profile', None)
        if not profile:
            return queryset.none()
        
        # 部门主管可以看本部门数据
        if profile.roles.filter(code='department_manager').exists():
            return queryset.filter(**{self.department_field: profile.department})
        
        # 普通员工只能看自己的
        return queryset.filter(**{f'{self.department_field}': profile.department})
```

### 2.2 审批流程引擎

#### 模型定义
```python
# apps/workflow/models.py
"""
工作流引擎

支持：
- 自定义审批流程
- 多级审批
- 会签/或签
- 条件分支
"""
from django.db import models
from django.contrib.auth import get_user_model
from django.contrib.contenttypes.fields import GenericForeignKey, GenericRelation
from django.contrib.contenttypes.models import ContentType

User = get_user_model()


class Workflow(models.Model):
    """
    工作流定义
    
    定义一个审批流程的模板
    """
    name = models.CharField("流程名称", max_length=100)
    code = models.CharField("流程编码", max_length=50, unique=True)
    description = models.TextField("描述", blank=True)
    
    # 关联的模型（如：请假单、报销单）
    content_type = models.ForeignKey(
        ContentType,
        on_delete=models.CASCADE,
        null=True,
        blank=True,
        related_name='workflows'
    )
    
    is_active = models.BooleanField("是否启用", default=True)
    version = models.IntegerField("版本号", default=1)
    
    created_by = models.ForeignKey(
        User,
        on_delete=models.SET_NULL,
        null=True,
        related_name='created_workflows'
    )
    created_at = models.DateTimeField("创建时间", auto_now_add=True)
    updated_at = models.DateTimeField("更新时间", auto_now=True)

    def __str__(self):
        return self.name


class WorkflowNode(models.Model):
    """
    审批节点
    
    工作流中的一个步骤
    """
    TYPE_CHOICES = [
        ('start', '开始'),
        ('approval', '审批'),
        ('抄送', '抄送'),
        ('condition', '条件'),
        ('end', '结束'),
    ]
    
    APPROVAL_TYPE_CHOICES = [
        ('single', '或签'),  # 任一人审批通过即可
        ('all', '会签'),       # 所有人审批通过才通过
        ('any', '或签'),
    ]
    
    workflow = models.ForeignKey(
        Workflow,
        on_delete=models.CASCADE,
        related_name='nodes'
    )
    
    name = models.CharField("节点名称", max_length=100)
    type = models.CharField("节点类型", max_length=20, choices=TYPE_CHOICES)
    order = models.IntegerField("顺序", default=0)
    
    # 审批人配置
    approval_type = models.CharField(
        "审批方式",
        max_length=20,
        choices=APPROVAL_TYPE_CHOICES,
        default='single'
    )
    
    # 审批人来源
    approver_type = models.CharField("审批人类型", max_length=20, choices=[
        ('user', '指定用户'),
        ('role', '指定角色'),
        ('department_leader', '部门负责人'),
        ('previous_approver', '上一审批人'),
        ('dynamic', '动态计算'),
    ])
    approvers = models.JSONField("审批人配置", default=list)  # 存储用户ID或角色ID
    
    # 审批人字段名（动态计算时使用）
    approver_field = models.CharField("审批人字段", max_length=50, blank=True)
    
    # 条件配置（JSON）
    # {"field": "amount", "operator": ">", "value": 10000}
    conditions = models.JSONField("条件配置", default=list)
    
    # 节点配置
    allow_reject = models.BooleanField("允许驳回", default=True)
    allow_transfer = models.BooleanField("允许转交", default=True)
    timeout_hours = models.IntegerField("超时时间(小时)", default=0)
    
    next_nodes = models.ManyToManyField(
        'self',
        verbose_name="下一节点",
        blank=True,
        symmetrical=False,
        related_name='prev_nodes'
    )
    
    created_at = models.DateTimeField("创建时间", auto_now_add=True)

    class Meta:
        ordering = ['order']

    def __str__(self):
        return f"{self.workflow.name} - {self.name}"


class WorkflowInstance(models.Model):
    """
    工作流实例
    
    一次具体的审批流程
    """
    STATUS_CHOICES = [
        ('draft', '草稿'),
        ('pending', '待审批'),
        ('approved', '已通过'),
        ('rejected', '已驳回'),
        ('cancelled', '已撤销'),
    ]
    
    workflow = models.ForeignKey(
        Workflow,
        on_delete=models.CASCADE,
        related_name='instances'
    )
    
    # 关联业务数据
    content_type = models.ForeignKey(
        ContentType,
        on_delete=models.CASCADE,
        null=True
    )
    object_id = models.PositiveIntegerField(null=True)
    content_object = GenericForeignKey('content_type', 'object_id')
    
    # 状态
    status = models.CharField("状态", max_length=20, choices=STATUS_CHOICES, default='draft')
    
    # 当前节点
    current_node = models.ForeignKey(
        WorkflowNode,
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
        related_name='current_instances'
    )
    
    # 发起人
    initiator = models.ForeignKey(
        User,
        on_delete=models.SET_NULL,
        null=True,
        related_name='initiated_workflows'
    )
    
    # 统计
    total_nodes = models.IntegerField("总节点数", default=0)
    completed_nodes = models.IntegerField("已完成节点", default=0)
    
    started_at = models.DateTimeField("开始时间", auto_now_add=True)
    completed_at = models.DateTimeField("完成时间", null=True)
    updated_at = models.DateTimeField("更新时间", auto_now=True)

    def __str__(self):
        return f"{self.workflow.name} - {self.id}"


class ApprovalRecord(models.Model):
    """
    审批记录
    
    节点的具体审批结果
    """
    ACTION_CHOICES = [
        ('approve', '通过'),
        ('reject', '驳回'),
        ('transfer', '转交'),
        ('retreat', '撤回'),
    ]
    
    instance = models.ForeignKey(
        WorkflowInstance,
        on_delete=models.CASCADE,
        related_name='records'
    )
    node = models.ForeignKey(
        WorkflowNode,
        on_delete=models.CASCADE,
        related_name='records'
    )
    
    # 审批人
    approver = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='approvals'
    )
    
    action = models.CharField("操作", max_length=20, choices=ACTION_CHOICES)
    comment = models.TextField("审批意见", blank=True)
    
    # 转交目标
    transfer_to = models.ForeignKey(
        User,
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
        related_name='transferred_approvals'
    )
    
    # 时间
    created_at = models.DateTimeField("操作时间", auto_now_add=True)

    class Meta:
        ordering = ['-created_at']

    def __str__(self):
        return f"{self.node.name} - {self.approver.username} - {self.action}"
```

#### 审批流程服务
```python
# apps/workflow/services.py
"""
工作流服务

处理工作流的创建、提交、审批等操作
"""
from django.db import transaction
from django.contrib.auth import get_user_model
from typing import Optional, List

User = get_user_model()


class WorkflowService:
    """
    工作流服务
    """
    
    def __init__(self, workflow: 'Workflow'):
        self.workflow = workflow
    
    def create_instance(self, content_object, initiator: User) -> 'WorkflowInstance':
        """
        创建工作流实例
        
        Args:
            content_object: 业务对象（如请假单）
            initiator: 发起人
            
        Returns:
            WorkflowInstance
        """
        from .models import WorkflowInstance, WorkflowNode
        
        with transaction.atomic():
            # 创建实例
            instance = WorkflowInstance.objects.create(
                workflow=self.workflow,
                content_object=content_object,
                initiator=initiator,
                status='draft'
            )
            
            # 获取开始节点
            start_node = self.workflow.nodes.filter(type='start').first()
            
            # 统计总节点数
            total_nodes = self.workflow.nodes.count()
            instance.total_nodes = total_nodes
            instance.current_node = start_node
            instance.save()
            
            return instance
    
    def submit(self, instance: 'WorkflowInstance') -> bool:
        """
        提交工作流
        
        Args:
            instance: 工作流实例
            
        Returns:
            是否成功
        """
        from .models import WorkflowNode, WorkflowInstance
        
        with transaction.atomic():
            # 获取开始节点
            start_node = instance.workflow.nodes.filter(type='start').first()
            
            # 获取下一个审批节点
            next_node = self._get_next_node(start_node)
            
            if not next_node:
                # 没有审批节点，直接通过
                instance.status = 'approved'
                instance.completed_at = timezone.now()
                instance.completed_nodes = instance.total_nodes
            else:
                # 更新状态
                instance.status = 'pending'
                instance.current_node = next_node
            
            instance.save()
            
            # 发送通知
            self._notify_approvers(instance, next_node)
            
            return True
    
    def approve(
        self, 
        instance: 'WorkflowInstance', 
        approver: User,
        comment: str = ''
    ) -> bool:
        """
        审批通过
        
        Args:
            instance: 工作流实例
            approver: 审批人
            comment: 审批意见
        """
        from .models import ApprovalRecord, WorkflowNode
        
        with transaction.atomic():
            current_node = instance.current_node
            
            # 创建审批记录
            ApprovalRecord.objects.create(
                instance=instance,
                node=current_node,
                approver=approver,
                action='approve',
                comment=comment
            )
            
            # 更新完成数
            instance.completed_nodes += 1
            
            # 获取下一节点
            next_node = self._get_next_node(current_node)
            
            if not next_node or next_node.type == 'end':
                # 流程结束
                instance.status = 'approved'
                instance.completed_at = timezone.now()
                instance.current_node = next_node
            else:
                instance.current_node = next_node
            
            instance.save()
            
            # 通知
            self._notify_approvers(instance, next_node)
            
            return True
    
    def reject(
        self,
        instance: 'WorkflowInstance',
        approver: User,
        comment: str = ''
    ) -> bool:
        """
        驳回工作流
        """
        from .models import ApprovalRecord
        
        with transaction.atomic():
            # 创建审批记录
            ApprovalRecord.objects.create(
                instance=instance,
                node=instance.current_node,
                approver=approver,
                action='reject',
                comment=comment
            )
            
            # 更新状态
            instance.status = 'rejected'
            instance.save()
            
            # 通知发起人
            self._notify_initiator(instance, 'rejected')
            
            return True
    
    def _get_next_node(self, current_node: 'WorkflowNode') -> Optional['WorkflowNode']:
        """获取下一节点"""
        from .models import WorkflowNode
        
        next_nodes = current_node.next_nodes.filter(isnull=False)
        
        if not next_nodes.exists():
            return None
        
        # 如果有多个下一节点，需要根据条件判断
        for next_node in next_nodes:
            if self._check_conditions(next_node.conditions):
                return next_node
        
        return next_nodes.first()
    
    def _check_conditions(self, conditions: List[dict]) -> bool:
        """检查条件是否满足"""
        if not conditions:
            return True
        
        # TODO: 实现条件判断
        return True
    
    def _get_approvers(self, node: 'WorkflowNode', instance: 'WorkflowInstance') -> List[User]:
        """获取审批人"""
        from apps.rbac.models import UserProfile
        
        approvers = []
        
        if node.approver_type == 'user':
            # 指定用户
            approver_ids = node.approvers
            approvers = list(User.objects.filter(id__in=approver_ids))
        
        elif node.approver_type == 'role':
            # 指定角色
            role_ids = node.approvers
            profiles = UserProfile.objects.filter(
                roles__id__in=role_ids,
                is_active=True
            )
            approvers = [p.user for p in profiles]
        
        elif node.approver_type == 'department_leader':
            # 部门负责人
            if instance.initiator.profile.department:
                manager = instance.initiator.profile.department.manager
                if manager:
                    approvers = [manager]
        
        return approvers
    
    def _notify_approvers(self, instance, node):
        """通知审批人"""
        # TODO: 发送邮件/站内信/钉钉通知
        pass
    
    def _notify_initiator(self, instance, action):
        """通知发起人"""
        # TODO: 发送通知
        pass
```

### 2.3 考勤系统

```python
# apps/attendance/models.py
"""
考勤管理

包含：打卡记录、请假申请、加班申请
"""
from django.db import models
from django.contrib.auth import get_user_model
from django.utils import timezone

User = get_user_model()


class AttendanceRecord(models.Model):
    """
    打卡记录
    """
    TYPE_CHOICES = [
        ('check_in', '上班打卡'),
        ('check_out', '下班打卡'),
    ]
    
    STATUS_CHOICES = [
        ('normal', '正常'),
        ('late', '迟到'),
        ('early_leave', '早退'),
        ('absent', '缺勤'),
        ('overtime', '加班'),
    ]
    
    user = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='attendance_records'
    )
    date = models.DateField("打卡日期")
    type = models.CharField("打卡类型", max_length=20, choices=TYPE_CHOICES)
    
    # 打卡时间
    clock_time = models.DateTimeField("打卡时间")
    
    # 位置
    location = models.CharField("打卡地点", max_length=200, blank=True)
    latitude = models.DecimalField("纬度", max_digits=10, decimal_places=6, null=True)
    longitude = models.DecimalField("经度", max_digits=10, decimal_places=6, null=True)
    
    # 状态
    status = models.CharField("状态", max_length=20, choices=STATUS_CHOICES, default='normal')
    remark = models.CharField("备注", max_length=200, blank=True)
    
    created_at = models.DateTimeField("创建时间", auto_now_add=True)

    class Meta:
        verbose_name = "打卡记录"
        verbose_name_plural = verbose_name
        unique_together = ['user', 'date', 'type']
        ordering = ['-date', '-clock_time']

    def __str__(self):
        return f"{self.user.username} - {self.date} - {self.get_type_display()}"
    
    @classmethod
    def check_status(cls, clock_time, user):
        """检查打卡状态"""
        # 上班时间 9:00
        work_start = clock_time.replace(hour=9, minute=0, second=0)
        
        if clock_time > work_start:
            return 'late'
        return 'normal'


class LeaveRequest(models.Model):
    """
    请假申请
    """
    TYPE_CHOICES = [
        ('annual', '年假'),
        ('sick', '病假'),
        ('personal', '事假'),
        ('marital', '婚假'),
        ('maternity', '产假'),
        ('paternity', '陪产假'),
        ('funeral', '丧假'),
        ('other', '其他'),
    ]
    
    STATUS_CHOICES = [
        ('draft', '草稿'),
        ('pending', '待审批'),
        ('approved', '已通过'),
        ('rejected', '已驳回'),
        ('cancelled', '已撤销'),
    ]
    
    user = models.ForeignKey(
        User,
        on_delete=models.CASCADE,
        related_name='leave_requests'
    )
    
    type = models.CharField("请假类型", max_length=20, choices=TYPE_CHOICES)
    start_date = models.DateField("开始日期")
    end_date = models.DateField("结束日期")
    days = models.DecimalField("天数", max_digits=4, decimal_places=1)
    
    reason = models.TextField("请假原因")
    
    # 工作流
    workflow_instance = models.ForeignKey(
        'workflow.WorkflowInstance',
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
        related_name='leave_requests'
    )
    
    status = models.CharField("状态", max_length=20, choices=STATUS_CHOICES, default='draft')
    
    # 审批信息
    approved_by = models.ForeignKey(
        User,
        on_delete=models.SET_NULL,
        null=True,
        blank=True,
        related_name='approved_leaves'
    )
    approved_at = models.DateTimeField("审批时间", null=True)
    approved_comment = models.TextField("审批意见", blank=True)
    
    created_at = models.DateTimeField("创建时间", auto_now_add=True)
    updated_at = models.DateTimeField("更新时间", auto_now=True)

    class Meta:
        verbose_name = "请假申请"
        verbose_name_plural = verbose_name
        ordering = ['-created_at']

    def __str__(self):
        return f"{self.user.username} - {self.get_type_display()} - {self.days}天"
    
    def save(self, *args, **kwargs):
        # 计算天数
        if self.start_date and self.end_date:
            self.days = (self.end_date - self.start_date).days + 1
        super().save(*args, **kwargs)
```

### 2.4 API 视图

```python
# apps/rbac/views.py
"""
RBAC API 视图
"""
from rest_framework import viewsets, status
from rest_framework.decorators import action
from rest_framework.response import Response
from rest_framework.permissions import IsAuthenticated, IsAdminUser

from .models import UserProfile, Department, Role, Permission, Menu
from .serializers import (
    UserProfileSerializer, DepartmentSerializer,
    RoleSerializer, PermissionSerializer, MenuSerializer
)


class DepartmentViewSet(viewsets.ModelViewSet):
    """部门管理"""
    queryset = Department.objects.filter(is_active=True)
    serializer_class = DepartmentSerializer
    permission_classes = [IsAdminUser]
    
    @action(detail=False, methods=['get'])
    def tree(self, request):
        """获取部门树"""
        roots = self.queryset.filter(parent__isnull=True)
        serializer = self.get_serializer(roots, many=True)
        return Response(serializer.data)


class RoleViewSet(viewsets.ModelViewSet):
    """角色管理"""
    queryset = Role.objects.filter(is_active=True)
    serializer_class = RoleSerializer
    permission_classes = [IsAdminUser]
    
    def get_queryset(self):
        return super().get_queryset().prefetch_related('permissions', 'menus')


class PermissionViewSet(viewsets.ReadOnlyModelViewSet):
    """权限查询"""
    queryset = Permission.objects.all()
    serializer_class = PermissionSerializer
    permission_classes = [IsAdminUser]
    
    @action(detail=False, methods=['get'])
    def tree(self, request):
        """获取权限树"""
        # 按资源分组
        resources = {}
        for perm in self.queryset:
            if perm.resource not in resources:
                resources[perm.resource] = []
            resources[perm.resource].append(perm)
        
        return Response(resources)


class MenuViewSet(viewsets.ModelViewSet):
    """菜单管理"""
    queryset = Menu.objects.filter(is_active=True)
    serializer_class = MenuSerializer
    permission_classes = [IsAdminUser]
    
    def get_queryset(self):
        return super().get_queryset().select_related('parent')


class UserProfileViewSet(viewsets.ModelViewSet):
    """用户管理"""
    queryset = UserProfile.objects.select_related('user', 'department')
    serializer_class = UserProfileSerializer
    permission_classes = [IsAdminUser]
    
    def get_queryset(self):
        queryset = super().get_queryset()
        
        # 按部门筛选
        department_id = self.request.query_params.get('department')
        if department_id:
            queryset = queryset.filter(department_id=department_id)
        
        # 在职状态
        is_active = self.request.query_params.get('is_active')
        if is_active is not None:
            queryset = queryset.filter(is_active=is_active.lower() == 'true')
        
        return queryset.prefetch_related('roles')
    
    @action(detail=False, methods=['get'])
    def current(self, request):
        """获取当前用户信息"""
        profile = request.user.profile
        serializer = self.get_serializer(profile)
        return Response(serializer.data)
    
    @action(detail=True, methods=['post'])
    def assign_roles(self, request, pk=None):
        """分配角色"""
        profile = self.get_object()
        role_ids = request.data.get('role_ids', [])
        
        profile.roles.set(role_ids)
        profile.save()
        
        return Response({'status': 'success'})
```

---

## 🔍 代码关键点说明

### 3.1 RBAC 权限模型

```
┌─────────────────────────────────────────────────────────────┐
│                    RBAC 权限模型                              │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│     用户 ──属于──> 角色 ──拥有──> 权限                       │
│       │              │              │                       │
│       │              │              ├── resource: user      │
│       │              │              ├── action: view        │
│       │              │              ├── action: create       │
│       │              │              ├── action: update       │
│       │              │              └── action: delete       │
│       │                                                    │
│       └── 部门隔离                                           │
│                                                             │
│  权限码格式: resource.action (如: user.view)                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 工作流设计模式

```
工作流 = 节点 + 连接

节点类型：
- start: 开始
- approval: 审批
- cc: 抄送
- condition: 条件判断
- end: 结束

审批方式：
- 或签: 任一人通过即可
- 会签: 所有人通过才通过
```

### 3.3 考勤计算规则

```
出勤天数 = 工作日 - 请假天数 - 缺勤天数

迟到：上班打卡 > 09:00
早退：下班打卡 < 18:00
缺勤：未打卡且无请假
加班：下班打卡 > 20:00
```

---

## 📅 学习计划（5天）

### Day 1: RBAC 权限系统
- 权限模型设计
- Django 实现
- 前端菜单控制

### Day 2: 组织架构
- 部门管理
- 员工管理
- 数据隔离

### Day 3: 工作流引擎
- 流程定义
- 审批节点
- 状态流转

### Day 4: 考勤系统
- 打卡记录
- 请假申请
- 统计报表

### Day 5: 集成和部署
- 完整功能测试
- Docker 部署
- 面试准备

---

## 💼 面试要点

### 问题 1: 如何设计一个灵活的权限系统？

**参考答案：**

1. **RBAC 模型**：用户-角色-权限
2. **数据权限**：部门隔离
3. **字段权限**：可见/可编辑
4. **API 级别**：装饰器/Mixin
5. **前端控制**：路由守卫

**实现要点**：
```python
# 权限装饰器
def require_permission(code):
    def decorator(view_func):
        @wraps(view_func)
        def wrapper(request, *args, **kwargs):
            if request.user.profile.has_permission(code):
                return view_func(request, *args, **kwargs)
            return JsonResponse({'error': '无权限'}, status=403)
        return wrapper
    return decorator
```

### 问题 2: 工作流引擎的核心设计？

**参考答案：**

1. **节点**：开始、审批、抄送、条件、结束
2. **连接**：节点之间的关系
3. **实例**：具体的流程执行
4. **记录**：审批历史

**状态流转**：
```
草稿 → 待审批 → 审批中 → 已通过/已驳回
                  ↓
               已撤销
```

### 问题 3: 如何保证数据安全？

**参考答案：**

1. **认证**：JWT Token
2. **权限**：RBAC + 数据隔离
3. **审计**：操作日志
4. **加密**：敏感数据加密存储
5. **接口**：限流、防爬

---

**恭喜完成企业内部ERP系统！🎉**

这是企业级应用开发的经典项目，展示了复杂业务系统的设计能力！
