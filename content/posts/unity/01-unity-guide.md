---
title: Unity引导实现方案
subtitle:
date: 2024-12-16T12:24:32+08:00
slug: f04284d
draft: false
author:
  name: mzbswh
  link:
  email:
  avatar:
description:
keywords:
license:
comment: false
weight: 0
tags:
  - unity
  - guide
categories:
  - unity
  - misc
hiddenFromHomePage: false
hiddenFromSearch: false
hiddenFromRelated: false
hiddenFromFeed: false
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
math: false
lightgallery: false
password:
message:
repost:
  enable: true
  url:

# See details front matter: https://fixit.lruihao.cn/documentation/content-management/introduction/#front-matter
---

> [!note] 简介
> 游戏的引导实现一直是比较令人头疼的问题，本文将实现一种较为通用的unity引导实现方案。包括引导逻辑驱动，挖洞遮罩等。

## 1. 引导基本逻辑 

引导基本逻辑可以划分为两个部分：引导流程和具体引导步骤

### 1.1 引导步骤

引导步骤是引导的最小单元，如一次点击操作。

```c#
public abstract class GuideStepBase
{
    /// <summary> 所属引导流程 </summary>
    public GuideProcedureBase GuideProcedure;
    /// <summary> 引导步骤Id: 引导流程内唯一, 100为基数 </summary>
    public int StepId { get; private set; }
    /// <summary> 步骤名称 </summary>、
    public string StepName { get; private set; }
    /// <summary> 是否是强制引导 </summary>
    public bool ForceGuide = true;
    /// <summary> 引导步骤状态 </summary>
    public GuideStepState State { get; protected set; } = GuideStepState.Leave;
    /// <summary> 下一个引导步骤 </summary>
    public int NextStepId { get; private set; }
    /// <summary> 加载时如果是此步骤需要跳转的步骤 </summary>
    public int JumpStepOnLoad { get; private set; }
    /// <summary> 是否记录步骤 </summary>
    public bool SaveStep { get; private set; } = true;

    public bool IsRunning => State == GuideStepState.Running;
    public bool IsPause => State == GuideStepState.Pause;
    public bool IsStoped => State == GuideStepState.Stoped;
    public bool IsLeave => State == GuideStepState.Leave;
    public bool IsEnter => State == GuideStepState.Enter;
    public bool Started => IsRunning || IsPause;
    public bool NotStart => IsStoped || IsEnter;

    protected Action<GuideStepBase> _onEnter;                        // 进入回调
    protected Action<GuideStepBase> _onStart;                        // 开始回调
    protected Action<GuideStepBase> _onPause;                        // 暂停回调
    protected Action<GuideStepBase> _onResume;                       // 恢复回调
    protected Action<GuideStepBase> _onStop;                         // 停止回调
    protected Action<GuideStepBase> _onComplete;                     // 完成回调
    protected Action<GuideStepBase> _onLeave;                        // 离开回调
    protected Action<GuideStepBase, float> _onUpdate;                // 更新回调
    protected Action<GuideStepBase, string, object[]> _onHandleMsg;  // 消息回调
    protected Func<GuideStepBase, bool> _startCondition;             // 开始条件
    protected Func<GuideStepBase, bool> _extraStartCondition;        // 开始条件
    protected Func<GuideStepBase, RectTransform> _getGuideTarget;    // 开始条件

    public GuideStepBase(GuideProcedureBase guideProcedure, int stepId, string name = null)
    {
        GuideProcedure = guideProcedure;
        StepId = stepId;
        StepName = string.IsNullOrEmpty(name) ? stepId.ToString() : name;
    }

    // 进入此引导步骤
    public virtual void Enter()
    {
        // 等待开始
        if (!IsLeave) return;
        SetState(GuideStepState.Enter);
        _onEnter?.Invoke(this);
    }

    // 引导开始：引导的对象或者引导的条件准备好了
    public virtual void Start()
    {
        if (Started || IsLeave) return;
        SetState(GuideStepState.Running);
        _onStart?.Invoke(this);
    }

    // 引导停止：引导的对象或者引导的条件不满足了，此时引导可能还不算完成，没有离开
    public virtual void Stop()
    {
        if (NotStart || IsLeave) return;
        SetState(GuideStepState.Stoped);
        _onStop?.Invoke(this);
    }

    // 引导暂停：引导对象隐藏或者不可点击时，临时暂停引导
    public virtual void Pause()
    {
        if (IsStoped || IsPause || IsLeave) return;
        SetState(GuideStepState.Pause);
        _onPause?.Invoke(this);
    }

    // 引导恢复：引导对象显示或者可点击时，恢复引导
    public virtual void Resume()
    {
        if (IsStoped || IsRunning || IsLeave) return;
        SetState(GuideStepState.Running);
        _onResume?.Invoke(this);
    }

    // 离开此引导步骤
    public virtual void Leave()
    {
        if (IsLeave) return;
        if (Started)
        {
            Stop();
        }
        SetState(GuideStepState.Leave);
        _onLeave?.Invoke(this);
    }

    // Update
    public virtual void Update(float deltaTime)
    {
        _onUpdate?.Invoke(this, deltaTime);
    }

    // 设置引导步骤完成
    public virtual void SetComplete()
    {
        if (IsLeave) return;
        _onComplete?.Invoke(this);
        GuideProcedure.HandleGuideStepComplete(this);
    }

    public GuideStepBase SetNextStep(GuideStepBase next)
    {
        NextStepId = next.StepId;
        return this;
    }

    public GuideStepBase SetNextStep(int stepId)
    {
        NextStepId = stepId;
        return this;
    }

    public GuideStepBase SetJumpStepOnLoad(int jumpStepOnLoad)
    {
        JumpStepOnLoad = jumpStepOnLoad;
        return this;
    }

    public GuideStepBase SetSaveStep(bool save)
    {
        SaveStep = save;
        return this;
    }

    // 处理消息
    public virtual void HandleMsg(string msg, params object[] args)
    {
        _onHandleMsg?.Invoke(this, msg, args);
    }

    // 是否能开始引导
    public virtual bool CanStart()
    {
        if (_extraStartCondition != null && !_extraStartCondition(this))
        {
            return false;
        }
        if (_startCondition == null)
        {
            var rectTf = GetGuideTargetRectTf();
            return (rectTf != null) && rectTf.gameObject.activeInHierarchy && rectTf.gameObject.activeSelf;
        }
        return _startCondition(this);
    }

    // 设置引导步骤状态
    protected virtual void SetState(GuideStepState state)
    {
        if (State == state) return;
        if (state == GuideStepState.Running)
        {
            GameplayUtil.ViewIns?.PauseGame();
        }
        else if (State == GuideStepState.Running)
        {
            // 原来是运行状态，现在不是了
            GameplayUtil.ViewIns?.ResumeGame();
        }
        State = state;
    }

    public RectTransform GetGuideTargetRectTf()
    {
        if (_getGuideTarget != null)
        {
            return _getGuideTarget(this);
        }
        return OnGetGuideTargetRectTf();
    }

    protected virtual RectTransform OnGetGuideTargetRectTf() => null;

    // callback
    public GuideStepBase OnEnter(Action<GuideStepBase> cb) { _onEnter = cb; return this; }
    public GuideStepBase OnStart(Action<GuideStepBase> cb) { _onStart = cb; return this; }
    public GuideStepBase OnPause(Action<GuideStepBase> cb) { _onPause = cb; return this; }
    public GuideStepBase OnResume(Action<GuideStepBase> cb) { _onResume = cb; return this; }
    public GuideStepBase OnStop(Action<GuideStepBase> cb) { _onStop = cb; return this; }
    public GuideStepBase OnComplete(Action<GuideStepBase> cb) { _onComplete = cb; return this; }
    public GuideStepBase OnLeave(Action<GuideStepBase> cb) { _onLeave = cb; return this; }
    public GuideStepBase OnUpdate(Action<GuideStepBase, float> cb) { _onUpdate = cb; return this; }
    public GuideStepBase OnHandleMsg(Action<GuideStepBase, string, object[]> cb) { _onHandleMsg = cb; return this; }
    public GuideStepBase SetStartCondition(Func<GuideStepBase, bool> cb) { _startCondition = cb; return this; }
    public GuideStepBase SetExtraStartCondition(Func<GuideStepBase, bool> cb) { _extraStartCondition = cb; return this; }
    public GuideStepBase SetGetGuideTarget(Func<GuideStepBase, RectTransform> cb) { _getGuideTarget = cb; return this; }
}

public enum GuideStepState
{
    Enter,
    Running,
    Pause,
    Stoped,
    Leave
}
```

### 1.2 引导流程

引导流程是一个完整的引导过程，包含多个引导步骤。

```c#
public abstract class GuideProcedureBase
{
    /// <summary> 引导流程配置Id </summary>
    public int Id { get; private set; }
    /// <summary> 引导流程名称 </summary>
    public virtual string Name => Id.ToString();
    /// <summary> 引导流程优先级 </summary>
    public int Priotity { get; private set; }
    /// <summary> 引导是否正在运行 </summary>
    public bool Running { get; private set; }
    /// <summary> 引导流程是否已完成 </summary>
    public bool Completed { get; private set; }
    /// <summary> 当前引导步骤 </summary>
    public GuideStepBase CurrentGuideStep { get; private set; }
    /// <summary> 是否有引导步骤在运行 </summary>
    public bool CurrentHasGuideStep => CurrentGuideStep != null;
    /// <summary> 是否正在引导中 </summary>
    public bool InGuiding => CurrentHasGuideStep && CurrentGuideStep.IsRunning;
    /// <summary> 是否正在强引导中 </summary>
    public bool InForceGuidiing => InGuiding && CurrentGuideStep.ForceGuide;
    /// <summary> 当前引导步骤Id（小于0说明没引导） </summary>
    public int CurrentGuideStepId => CurrentGuideStep?.StepId ?? -1;
    /// <summary> 首个引导步骤 </summary>
    public virtual GuideStepBase FirstGuideStep { get; protected set; }

    /// <summary> 引导步骤字典：key=引导步骤Id </summary>
    private readonly Dictionary<int, GuideStepBase> _guideStepMap = new();

    public GuideProcedureBase(int id, int priority = 0)
    {
        Id = id;
        Running = false;
        Priotity = priority;
        InitGuideSteps();
    }

    /// <summary> 能否开始引导流程 </summary>
    public virtual bool CanStart() => true;

    /// <summary> 是否应该直接完成 </summary>
    public virtual bool ShouldComplete() => false;

    /// <summary> 获取引导步骤实例 </summary>
    public GuideStepBase GetGuideStep(int stepId) => _guideStepMap.GetOrDefault(stepId);

    public virtual void Update(float deltaTime)
    {
        if (!Running || !CurrentHasGuideStep) return;
        if (CurrentGuideStep.NotStart)
        {
            CheckCurrentStepStart();
        }
        CurrentGuideStep?.Update(deltaTime);
    }

    public void HandleGuideStepComplete(GuideStepBase guideStep)
    {
        if (!CurrentHasGuideStep || (CurrentGuideStep != guideStep)) return;
        LeaveStep(guideStep);
        CurrentGuideStep = null;
        EnterStep(guideStep.NextStepId);
        if (!CurrentHasGuideStep)
        {
            SetComplete(true);
        }
    }

    public virtual void ResetGuide()
    {
        Completed = false;
        LeaveStep(CurrentGuideStep);
        CurrentGuideStep = null;
    }
    
    public void Start()
    {
        if (!CurrentHasGuideStep || InGuiding) return;
        EnterStep(CurrentGuideStep);
    }

    public void Interrupt()
    {
        CurrentGuideStep?.Stop();
    }

    public virtual void SetComplete(bool checkGuide)
    {
        if (Completed) return;
        Completed = true;
        LeaveStep(CurrentGuideStep);
        CurrentGuideStep = null;
    }

    public void CheckCurrentStepStart()
    {
        if (!CurrentHasGuideStep || CurrentGuideStep.Started) return;
        if (!CurrentGuideStep.CanStart()) return;
        if (CurrentGuideStep.ForceGuide)
        {
            if (!CheckForceGuidePriority()) return; // 优先级低先不引导
            InterruptOtherForceGuding();
        }
        CurrentGuideStep.Start();
    }

    public void CheckCurrentStepResume()
    {
        if (!CurrentHasGuideStep || !CurrentGuideStep.IsPause) return;
        if (CurrentGuideStep.ForceGuide)
        {
            if (!CheckForceGuidePriority()) return; // 优先级低先不引导
            InterruptOtherForceGuding();
        }
        CurrentGuideStep.Resume();
    }

    public bool CheckForceGuidePriority()
    {
        var forceGuiding = GuideMgr.Ins.GetCurentForceGuiding();
        if (forceGuiding == null) return true;
        // 判断优先级
        if (forceGuiding.Priotity >= Priotity)
        {
            return false;  // 优先级低先不引导
        }
        return true;
    }

    public void InterruptOtherForceGuding()
    {
        var forceGuiding = GuideMgr.Ins.GetCurentForceGuiding();
        if ((forceGuiding == null) || (forceGuiding == this)) return;
        forceGuiding.Interrupt();
    }

    public bool EnterStep(int stepId)
    {
        if (!_guideStepMap.TryGetValue(stepId, out var guideStep)) return false;
        return EnterStep(guideStep);
    }

    public bool EnterStep(GuideStepBase guideStep)
    {
        LeaveStep(CurrentGuideStep);
        CurrentGuideStep = guideStep;
        if (guideStep.SaveStep)
        {
            SaveGuide(guideStep.StepId);
        }
        if (guideStep.ForceGuide)
        {
            if (!CheckForceGuidePriority()) return false; // 优先级低先不引导
            InterruptOtherForceGuding();
        }
        Running = true;
        CurrentGuideStep.Enter();
        CheckCurrentStepStart();    // 检查是否能开始
        return true;
    }

    public void LeaveStep(GuideStepBase guideStep)
    {
        guideStep?.Leave();
        Running = false;
    }

    public void SetStepNextStep(int stepId, int nextStepId)
    {
        if (!_guideStepMap.TryGetValue(stepId, out var guideStep)) return;
        if (!_guideStepMap.TryGetValue(nextStepId, out var nextStep)) return;
        guideStep.SetNextStep(nextStep);
    }

    public void SetStepJumpOnLoad(int stepId, int jumpStepId, bool autoSetJumpLastToStep = false)
    {
        if (!_guideStepMap.TryGetValue(stepId, out var guideStep)) return;
        guideStep.SetJumpStepOnLoad(jumpStepId);
        if (autoSetJumpLastToStep)
        {
            SetStepNextStep(jumpStepId, stepId);
        }
    }

    public void HandleMsg(string msg, params object[] args)
    {
        if (!CurrentHasGuideStep) return;
        CurrentGuideStep.HandleMsg(msg, args);
    }

    private GuideStepBase _lastAdd;
    protected virtual void AddGuideStep(GuideStepBase guideStep, bool autoSetLastNext = true)
    {
        if (_guideStepMap.Count == 0)
        {
            FirstGuideStep = guideStep;
        }
        if (_guideStepMap.ContainsKey(guideStep.StepId))
        {
            LogHelper.Error($"引导流程:{Name}，添加了重复的引导步骤Id:{guideStep.StepName}");
            return;
        }
        _guideStepMap.Add(guideStep.StepId, guideStep);
        if (autoSetLastNext && _lastAdd != null)
        {
            _lastAdd.SetNextStep(guideStep);
        }
        _lastAdd = guideStep;
    }

    /// <summary> 初始化引导步骤 </summary>
    protected abstract void InitGuideSteps();

    // 返回要进行的引导步骤： 如果没有说明引导已完成
    protected virtual GuideStepBase GetGuideStepBySavedStepId(int savedStepId)
    {
        if (savedStepId < 0) return null;
        if (savedStepId == 0)
        {
            return FirstGuideStep;
        }
        // 先看是否能直接查到某个步骤
        if (!_guideStepMap.TryGetValue(savedStepId, out var guideStep))
        {
            // 如果没有找到，就找下一个大步骤: 默认起始步骤Id是100的倍数+1
            savedStepId = (savedStepId / 100 + 1) * 100 + 1;
            _guideStepMap.TryGetValue(savedStepId, out guideStep);
        }
        if (guideStep == null)
        {
            return null;
        }
        if (guideStep.JumpStepOnLoad < 0)
        {
            return null;
        }
        if (guideStep.JumpStepOnLoad > 0)
        {
            if (_guideStepMap.TryGetValue(guideStep.JumpStepOnLoad, out var jumpStep))
            {
                return jumpStep;
            }
        }
        return guideStep;
    }
```

## 2. 引导遮罩实现

```shaderlab
Shader "ImageWithHole"
{
    Properties
    {
        [PerRendererData] _MainTex ("Sprite Texture", 2D) = "white" {}
        _TintColor ("Tint", Color) = (1,1,1,1)

        _StencilComp ("Stencil Comparison", Float) = 8
        _Stencil ("Stencil ID", Float) = 0
        _StencilOp ("Stencil Operation", Float) = 0
        _StencilWriteMask ("Stencil Write Mask", Float) = 255
        _StencilReadMask ("Stencil Read Mask", Float) = 255

        _MaskType("Mask Type", Integer) = 1                                 // 1: 圆形 2: 矩形
        _Center("Center", vector) = (0, 0, 0, 0)
        _Radius("Radius", Float) = 100                                      // 圆半径
        _ClipData ("Clip Rect", Vector) = (100,0,0,0)                       // leftBottom.x, leftBottom.y, rightTop.x, rightTop.y
        _RoundCornerRadius("Round Corner Radius", Float) = 20               // 圆角半径
        _TransitionRange("Transition Range", Range(0.1, 100)) = 0.1         // 过渡范围
    }

    SubShader
    {
        Tags
        {
            "Queue"="Transparent"
            "IgnoreProjector"="True"
            "RenderType"="Transparent"
            "PreviewType"="Plane"
            "CanUseSpriteAtlas"="True"
        }

        Stencil
        {
            Ref [_Stencil]
            Comp [_StencilComp]
            Pass [_StencilOp]
            ReadMask [_StencilReadMask]
            WriteMask [_StencilWriteMask]
        }

        Cull Off
        Lighting Off
        ZWrite Off
        Blend SrcAlpha OneMinusSrcAlpha
        ColorMask RGBA

        Pass
        {
            Name "Default"
        CGPROGRAM
            #pragma vertex vert
            #pragma fragment frag
            #pragma target 2.0

            #include "UnityCG.cginc"
            #include "UnityUI.cginc"

            struct appdata_t
            {
                float4 vertex   : POSITION;
                float4 color    : COLOR;
                float2 texcoord : TEXCOORD0;
                UNITY_VERTEX_INPUT_INSTANCE_ID
            };

            struct v2f
            {
                float4 vertex   : SV_POSITION;
                fixed4 color    : COLOR;
                float2 texcoord  : TEXCOORD0;
                float4 worldPosition : TEXCOORD1;
                UNITY_VERTEX_OUTPUT_STEREO
            };

            fixed4 _TintColor;
            fixed4 _TextureSampleAdd;
            fixed4 _ClipData;
            float2 _Center;
            half _Radius;
            half _TransitionRange;
            int _MaskType;
            float _RoundCornerRadius;

            sampler2D _MainTex;

            fixed getCornerAlpha(fixed isCorner, float dis)
            {
                fixed inCorner = isCorner * step(dis, _RoundCornerRadius);     // 圆角半径外不裁剪
                fixed transition = (1 - isCorner) + isCorner * saturate((dis - (_RoundCornerRadius - _TransitionRange)) / _TransitionRange);
                transition = (1 - inCorner) + inCorner * transition;
                return transition;
            }

            float smoothStep(float edge0, float edge1, float x)
            {
                float t = saturate((x - edge0) / (edge1 - edge0));
                return t * t * (3.0 - 2.0 * t);
            }

            v2f vert(appdata_t v)
            {
                v2f OUT;
                UNITY_SETUP_INSTANCE_ID(v);
                UNITY_INITIALIZE_VERTEX_OUTPUT_STEREO(OUT);
                OUT.worldPosition = v.vertex;
                OUT.vertex = UnityObjectToClipPos(OUT.worldPosition);

                OUT.texcoord = v.texcoord;

                OUT.color = v.color * _TintColor;
                return OUT;
            }

            fixed4 frag(v2f i) : SV_Target
            {
                half4 color = (tex2D(_MainTex, i.texcoord) + _TextureSampleAdd) * i.color;
                
                fixed typeIsRound = step(_MaskType, 1) * step(1, _MaskType);
                fixed typeIsRectangle = (1 - typeIsRound) * step(_MaskType, 2) * step(2, _MaskType);
                
                // ----------圆形----------
                // 计算片元屏幕和目标中心位置的距离
                float dis = distance(i.vertex.xy, _Center.xy);
                // 是否在圆里
                fixed insideRound = step(dis, _Radius) * typeIsRound;
                // 计算过渡范围内的alpha值
                color.a *= (1 - insideRound) + insideRound * saturate((dis - (_Radius - _TransitionRange)) / _TransitionRange);

                // ----------矩形----------
                float2 roundCorner = float2(_RoundCornerRadius, _RoundCornerRadius);
                // 是否在矩形里
                fixed insideRect = UnityGet2DClipping(i.vertex.xy, _ClipData) * typeIsRectangle;
                // 左下角
                fixed isLBCorner = step(i.vertex.x - _ClipData.x, _RoundCornerRadius) * step(i.vertex.y - _ClipData.y, _RoundCornerRadius) * insideRect;
                dis = distance(i.vertex.xy, _ClipData.xy + roundCorner);
                color.a *= getCornerAlpha(isLBCorner, dis);
                // 左上角
                fixed isLTCorner = step(i.vertex.x - _ClipData.x, _RoundCornerRadius) * step(_ClipData.w - i.vertex.y, _RoundCornerRadius) * insideRect;
                dis = distance(i.vertex.xy, float2(_ClipData.x + _RoundCornerRadius, _ClipData.w - _RoundCornerRadius));
                color.a *= getCornerAlpha(isLTCorner, dis);
                // 右上角
                fixed isRTCorner = step(_ClipData.z - i.vertex.x, _RoundCornerRadius) * step(_ClipData.w - i.vertex.y, _RoundCornerRadius) * insideRect;
                dis = distance(i.vertex.xy, _ClipData.zw - roundCorner);
                color.a *= getCornerAlpha(isRTCorner, dis);
                // 右下角
                fixed isRBCorner = step(_ClipData.z - i.vertex.x, _RoundCornerRadius) * step(i.vertex.y - _ClipData.y, _RoundCornerRadius) * insideRect;
                dis = distance(i.vertex.xy, float2(_ClipData.z - _RoundCornerRadius, _ClipData.y + _RoundCornerRadius));
                color.a *= getCornerAlpha(isRBCorner, dis);
                // 其他
                fixed notCorner = (1 - (isLBCorner + isLTCorner + isRTCorner + isRBCorner)) * insideRect;
                float halfSizeX = (_ClipData.z - _ClipData.x) / 2;
                float halfSizeY = (_ClipData.w - _ClipData.y) / 2;
                half disCenterX = distance(i.vertex.x, (_ClipData.x + _ClipData.z) / 2);    // 和x轴中心的距离
                half disCenterY = distance(i.vertex.y, (_ClipData.y + _ClipData.w) / 2);    // 和y轴中心的距离
                half alphaX= saturate((disCenterX - (halfSizeX - _TransitionRange)) / _TransitionRange);
                half alphaY= saturate((disCenterY - (halfSizeY - _TransitionRange)) / _TransitionRange);
                color.a *= (1 - notCorner) + notCorner * max(alphaX, alphaY);

                clip (color.a - 0.001);
                return color;
            }
        ENDCG
        }
    }
}
```

## 3. 判断UI是否可点击

在需要点击的UI上添加如下脚本：

```c#
public class GuideTarget : MonoBehaviour
{
    public GuideStepBase GuideStep;
    public bool CheckCanClick = true;

    private PointerEventData _pointerData;
    private Camera _uiCamera;
    private List<RaycastResult> _tempResults;

    void Awake()
    {
        _pointerData = new PointerEventData(EventSystem.current);
        _uiCamera = GameRoot.UI.InstanceRoot.GetComponent<Canvas>().worldCamera;
        _tempResults = new();
    }

    void Update()
    {
        if ((GuideStep == null) || !GuideStep.Started)
        {
            // 销毁自身
            GameObject.Destroy(this);
            return;
        }
        if (!CheckCanClick) return;
        if (GuideStep.IsRunning)
        {
            if (!CanClick())
            {
                GuideStep.Pause();
            }
        }
        else if (GuideStep.IsPause)
        {
            if (CanClick())
            {
                GuideStep.GuideProcedure.CheckCurrentStepResume();
            }
        }
    }

    void OnEnable()
    {
        if (GuideStep == null) return;
        GuideStep.GuideProcedure.CheckCurrentStepResume();
    }

    void OnDisable()
    {
        if (GuideStep == null) return;
        GuideStep.Pause();
    }

    void OnDestroy()
    {
        if (GuideStep == null) return;
        GuideStep.Stop();
    }

    public bool CanClick()
    {
        if ((gameObject == null) || !gameObject.activeInHierarchy || !gameObject.activeSelf) return false;
        var rectTf = transform as RectTransform;
        var pos = rectTf.TransformPoint(rectTf.rect.center);
        var screenPos = _uiCamera.WorldToScreenPoint(pos);
        _pointerData.position = screenPos;
        EventSystem.current.RaycastAll(_pointerData, _tempResults);
        if (_tempResults.Count == 0) return false;
        GameObject checkGo = null;
        // 忽略引导遮罩
        for (int i = 0; i < _tempResults.Count; i++)
        {
            checkGo = _tempResults[i].gameObject;
            if (_tempResults[i].gameObject.GetComponent<GuideRaycaterFilter>() == null)
            {
                checkGo = _tempResults[i].gameObject;
                break;
            }
        }
        var go = ExecuteEvents.GetEventHandler<IPointerClickHandler>(checkGo);

        if (go == null) return false;
        if (go != gameObject) return false;
        return true;
    }
}
```

## 4. 点击穿透实现

```c#
public class GuideRaycaterFilter : MonoBehaviour, ICanvasRaycastFilter
{
    public Func<Vector2, bool> ValidFunc;

    public bool IsRaycastLocationValid(Vector2 sp, Camera eventCamera)
    {
        if (ValidFunc == null) return true;
        return ValidFunc(sp);
    }
}
```

如让点击穿透挖洞区域，可让ValidFunc为如下方法：
```c#
// 穿透挖洞过滤方法：点击挖洞外部有效
private bool ThroughHoleRaycastFilter(Vector2 screenPos)
{
    // _centerPos为挖洞中心位置，_radius为挖洞半径，_rectSize为挖洞矩形大小
    var centerScreenPos = UIPosToScreenPos(_centerPos);
    switch (_maskType)
    {
        case GuideMaskType.Round:
        {
            // 判断点击位置是否在圆形挖洞外：与圆心距离大于半径
            var dis = Vector2.Distance(screenPos, centerScreenPos);
            return dis > _radius;
        }
        case GuideMaskType.Rect:
        {
            var rectScreenSize = GameRoot.UI.UISizeToScreenSize(_rectSize);
            // 判断点击位置是否在矩形挖洞外：在矩形范围外
            return (screenPos.x < (centerScreenPos.x - rectScreenSize.x / 2)) ||
                   (screenPos.x > (centerScreenPos.x + rectScreenSize.x / 2)) ||
                   (screenPos.y < (centerScreenPos.y - rectScreenSize.y / 2)) ||
                   (screenPos.y > (centerScreenPos.y + rectScreenSize.y / 2));
        }
        default: return true;
    } 
}
```
