Utility Mod SFcore
=====================================

Background
^^^^^^^^^^
You can add custom achievements, charms and items without having a good understanding of the various aspects that run Hollow Knight and its mods. 
Therefore, this guide recommends and expects that you have already worked with the following topics:

* Understanding the structure of a basic mod.
* Adding embedded resources to a mod.
* Adding references to other mods.

Add custom achievements
^^^^^^^^^^^^^^^^^^^^^^^

1) Initialize the AchievementHelper with 1 custom achievement and empty sprites.

.. note::
    This step is the one where you can add your custom sprites. Simply load those instead of the :code:`new Sprite()`.

.. note::
    For the :code:`Convo`'s to work properly, you need the :code:`ModHooks.Instance.LanguageGetHook` similar to the Helper above, but only listening to your custom convo keys.

.. code-block:: c#

    public override void Initialize()
    {
        AchievementHelper.Initialize();
        AchievementHelper.Add("YourCustomAchievementKey", new Sprite(), "YourCustomLanguageConvo", "YourCustomLanguageConvo", false);
    }

2) Done! Now you can at some point in your mod have :code:`GameManager.instance.AwardAchievement("YourCustomAchievementKey");` to grant the player the achievement.

Add custom charms
^^^^^^^^^^^^^^^^^

The code for a CharmHelper_Example_ is also available.

1) Preparation of your save settings of your mod to hold data for charms.

.. code-block:: c#

    // insert default values here
    public List<bool> gotCharms = new List<bool>() { true };
    public List<bool> newCharms = new List<bool>() { false };
    public List<bool> equippedCharms = new List<bool>() { false };
    public List<int> charmCosts = new List<int>() { 1 };

2) Add the CharmHelper as a member of your main mod class

.. code-block:: c#

    //private Sprite testSprite;
    public CharmHelper charmHelper { get; private set; }

3) Initialize the :code:`CharmHelper` with a custom charm and empty sprite, also initialize the callbacks needed.

.. note::
    This step is the one where you can add your custom sprites. Simply load those instead of the :code:`new Sprite()`.

.. code-block:: c#

    public override void Initialize()
    {
        //loadResources();
        charmHelper = new CharmHelper();
        charmHelper.customCharms = 4;
        charmHelper.customSprites = new Sprite[] { new Sprite(), new Sprite(), new Sprite(), new Sprite() };
        //charmHelper.customSprites = new Sprite[] { testSprite, testSprite, testSprite, testSprite };

        initCallbacks();
    }

4) Initialize the callbacks needed.

.. code-block:: c#

    private void initCallbacks()
    {
        ModHooks.Instance.LanguageGetHook += OnLanguageGetHook;
        ModHooks.Instance.GetPlayerBoolHook += OnGetPlayerBoolHook;
        ModHooks.Instance.SetPlayerBoolHook += OnSetPlayerBoolHook;
        ModHooks.Instance.GetPlayerIntHook += OnGetPlayerIntHook;
        ModHooks.Instance.SetPlayerIntHook += OnSetPlayerIntHook;
    }

5) The callbacks for language should include

.. code-block:: c#

    if (key.StartsWith("CHARM_NAME_"))
    {
        int charmNum = int.Parse(key.Split('_')[2]);
        if (charmHelper.charmIDs.Contains(charmNum))
        {
            return "CHARM NAME";
        }
    }
    if (key.StartsWith("CHARM_DESC_"))
    {
        int charmNum = int.Parse(key.Split('_')[2]);
        if (charmHelper.charmIDs.Contains(charmNum))
        {
            return "CHARM DESC";
        }
    }

6) The callbacks for boolean checks should include

.. code-block:: c#

    private bool OnGetPlayerBoolHook(string target)
    {
        if (target.StartsWith("gotCharm_"))
        {
            int charmNum = int.Parse(target.Split('_')[1]);
            if (charmHelper.charmIDs.Contains(charmNum))
            {
                return Settings.gotCharms[charmHelper.charmIDs.IndexOf(charmNum)];
            }
        }
        if (target.StartsWith("newCharm_"))
        {
            int charmNum = int.Parse(target.Split('_')[1]);
            if (charmHelper.charmIDs.Contains(charmNum))
            {
                return Settings.newCharms[charmHelper.charmIDs.IndexOf(charmNum)];
            }
        }
        if (target.StartsWith("equippedCharm_"))
        {
            int charmNum = int.Parse(target.Split('_')[1]);
            if (charmHelper.charmIDs.Contains(charmNum))
            {
                return Settings.equippedCharms[charmHelper.charmIDs.IndexOf(charmNum)];
            }
        }
        return PlayerData.instance.GetBoolInternal(target);
    }
    private void OnSetPlayerBoolHook(string target, bool val)
    {
        if (target.StartsWith("gotCharm_"))
        {
            int charmNum = int.Parse(target.Split('_')[1]);
            if (charmHelper.charmIDs.Contains(charmNum))
            {
                Settings.gotCharms[charmHelper.charmIDs.IndexOf(charmNum)] = val;
                return;
            }
        }
        if (target.StartsWith("newCharm_"))
        {
            int charmNum = int.Parse(target.Split('_')[1]);
            if (charmHelper.charmIDs.Contains(charmNum))
            {
                Settings.newCharms[charmHelper.charmIDs.IndexOf(charmNum)] = val;
                return;
            }
        }
        if (target.StartsWith("equippedCharm_"))
        {
            int charmNum = int.Parse(target.Split('_')[1]);
            if (charmHelper.charmIDs.Contains(charmNum))
            {
                Settings.equippedCharms[charmHelper.charmIDs.IndexOf(charmNum)] = val;
                return;
            }
        }
        PlayerData.instance.SetBoolInternal(target, val);
    }

7) The callbacks for integer checks should include

.. code-block:: c#

    private int OnGetPlayerIntHook(string target)
    {
        if (target.StartsWith("charmCost_"))
        {
            int charmNum = int.Parse(target.Split('_')[1]);
            if (charmHelper.charmIDs.Contains(charmNum))
            {
                return Settings.charmCosts[charmHelper.charmIDs.IndexOf(charmNum)];
            }
        }
        return PlayerData.instance.GetIntInternal(target);
    }
    private void OnSetPlayerIntHook(string target, int val)
    {
        // We don't need other mods to adjust the cost of our charms, but it could be added if wanted
        PlayerData.instance.SetIntInternal(target, val);
    }

Add custom enviroment particles
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1) Initialize the :code:`EnviromentParticleHelper` with custom particles. We will use enviroment type 7, which has no sprites but the same audio as grass enviroment. As such we won't be adding custom audio.

.. note::
    This step is the one where you can add your custom sprites. Simply load those instead of the :code:`new Sprite()`.

.. code-block:: c#

    public override void Initialize()
    {
        //loadResources();

        EnviromentParticleHelper.Init();
        EnviromentParticleHelper.AddCustomDashEffectsHook += AddCustomDashEffectsHook;
    }
    private (int enviromentType, GameObject dashEffects) AddCustomDashEffectsHook(DashEffect self)
    {
        var prefab = self.dashGrass;
        var tmp = UObject.Instantiate(prefab, prefab.transform.parent);
        var tmpPSR = tmp.GetComponentInChildren<ParticleSystemRenderer>();
        var tmpPSR_M = tmpPSR.materials;
        tmpPSR_M[0].SetTexture("_MainTex", new Texture());
        return (7, tmp);
    }

2) Done! You can now have custom enviroment particles.

Add custom inventory items
^^^^^^^^^^^^^^^^^^^^^^^^^^

1) Initialize the :code:`ItemHelper` with custom items and empty sprites.

.. note::
    This step is the one where you can add your custom sprites. Simply load those instead of the :code:`new Sprite()`.

.. note::
    For the :code:`Convo`'s to work properly, you need the :code:`ModHooks.Instance.LanguageGetHook` similar to the Helper above, but only listening to your custom convo keys.

.. note::
    For the :code:`playerdataBool` to work properly, you need the :code:`ModHooks.Instance.GetPlayerBoolHook` & :code:`ModHooks.Instance.SetPlayerBoolHook` similar to the CharmHelper, but only listening to your custom bool key.

.. note::
    For the :code:`playerdataInt` to work properly, you need the :code:`ModHooks.Instance.GetPlayerIntHook` & :code:`ModHooks.Instance.SetPlayerIntHook` similar to the CharmHelper, but only listening to your custom int key.

.. code-block:: c#

    public override void Initialize()
    {
        // Normal Items, like the Kings Brand, Crystal Heart, etc.
        ItemHelper.AddNormalItem("YourUniqueStateName", new Sprite(), "YourCustomPlayerDataBool", "YourCustomLanguageConvo", "YourCustomLanguageConvo");

        // Counted Items, like Simple Keys, Rancid Eggs, etc.
        ItemHelper.AddCountedItem("YourUniqueStateName", new Sprite(), "YourCustomPlayerDataInt", "YourCustomLanguageConvo", "YourCustomLanguageConvo");

        // 1 2 Items, like the Map, Quill, but without Map and Quill
        SFCore.ItemHelper.AddOneTwoItem("YourUniqueStateName",
            new Sprite(), new Sprite(), // Sprites
            "YourCustomPlayerDataBool", "YourCustomPlayerDataBool", // PlayerData Bools
            "YourCustomLanguageConvo", "YourCustomLanguageConvo", // Name Convos
            "YourCustomLanguageConvo", "YourCustomLanguageConvo"); // Description Convos

        // 1 2 Both Items, like the Map, Quill and Map and Quill
        SFCore.ItemHelper.AddOneTwoBothItem("YourUniqueStateName",
            new Sprite(), new Sprite(), new Sprite(), // Sprites
            "YourCustomPlayerDataBool", "YourCustomPlayerDataBool", // PlayerData Bools
            "YourCustomLanguageConvo", "YourCustomLanguageConvo", "YourCustomLanguageConvo", // Name Convos
            "YourCustomLanguageConvo", "YourCustomLanguageConvo", "YourCustomLanguageConvo"); // Description Convos
    }

2) Done! You can now have custom Inventory Items.

Add custom title logos
^^^^^^^^^^^^^^^^^^^^^^

1) Initialize the :code:`TitleLogoHelper` with a logo spite.

.. note::
    This step is is being done best in the constructor of your mod class.

.. note::
    This step is the one where you can add your custom sprites. Simply load those instead of the :code:`new Sprite()`.

.. code-block:: c#

    private int LogoId = -1;
    public ModName()
    {
        TitleLogoHelper.Initialize();
        LogoId = TitleLogoHelper.AddLogo(new Sprite());
    }

2) Done! You can now have custom title logos.

Add custom menu styles
^^^^^^^^^^^^^^^^^^^^^^

1) Initialize the :code:`MenuStyleHelper` with a "custom" menu theme. We make an unused menu style avaiable and also center the gameobjects of that style.

.. note::
    This step is is being done best in the constructor of your mod class.

.. note::
    This can utilize custom logos.

.. code-block:: c#

    public ModName()
    {
        MenuStyleHelper.Initialize();
        MenuStyleHelper.AddMenuStyleHook += AddMyMenuStyle;
    }

    // this auto-generates, but you can leave the variable names out to save space
    private (string languageString, GameObject styleGo, int titleIndex, string unlockKey, string[] achievementKeys, MenuStyles.MenuStyle.CameraCurves cameraCurves, AudioMixerSnapshot musicSnapshot) AddMyMenuStyle(MenuStyles self)
    {
        GameObject menuStylesGo = self.gameObject;
        var radiantStyleGo = menuStylesGo.transform.GetChild(4).gameObject;

        var cameraCurves = new MenuStyles.MenuStyle.CameraCurves
        {
            saturation = 1.0f,
            redChannel = new AnimationCurve(),
            greenChannel = new AnimationCurve(),
            blueChannel = new AnimationCurve()
        };
        cameraCurves.redChannel.AddKey(new Keyframe(0f, 0f));
        cameraCurves.redChannel.AddKey(new Keyframe(1f, 1f));
        cameraCurves.greenChannel.AddKey(new Keyframe(0f, 0f));
        cameraCurves.greenChannel.AddKey(new Keyframe(1f, 1f));
        cameraCurves.blueChannel.AddKey(new Keyframe(0f, 0f));
        cameraCurves.blueChannel.AddKey(new Keyframe(1f, 1f));

        AudioMixerSnapshot audioSnapshot = self.styles[1].musicSnapshot.audioMixer.FindSnapshot("Normal");
        
        // Replace the -1 with a custom Logo ID if you want to
        return ("UI_MENU_STYLE_RADIANT", radiantStyleGo, -1, "", null, cameraCurves, audioSnapshot);
    }

2) Done! You can now have custom menu styles.

"Debugging" PlayMaker FSMs
^^^^^^^^^^^^^^^^^^^^^^^^^^

Get your fsm and add a log action.

.. code-block:: c#

    var fsm = gameObject.LocateMyFsm("FSM name here");
    fsm.AddAction("FSM state name here", new ActualLogAction() { text = "Log message here" });

FsmUtil
^^^^^^^

Similar in functionality as the one found int ModCommon, but also works with the HK 1.5 beta.

.. code-block:: c#

    var fsm = gameObject.LocateMyFsm("FSM name here");
    fsm.AddFsmState("New empty state name");
    fsm.AddTransition("from state", "event name", "to state"); // the state has to exist first
    fsm.AddGlobalTransition("global event name", "to state"); // the state has to exist first
    fsm.ChangeTransition("from state", "event name", "new to state"); // the state has to exist first
    fsm.GetState("FSM state name here"); // returns a state, if you ever need it
    fsm.CopyState("original state name", "name of new copied state"); // returns copied state
    fsm.GetAction<ActionType>("FSM state name here", 0); // returns action instance
    fsm.AddAction("FSM state name here", new FsmStateAction());
    fsm.InsertAction("FSM state name here", new FsmStateAction(), 0);
    fsm.RemoveAction("FSM state name here", 0);
    fsm.AddFloatVariable("new variable name");
    fsm.AddIntVariable("new variable name");
    fsm.AddBoolVariable("new variable name");
    fsm.AddStringVariable("new variable name");
    fsm.AddVector2Variable("new variable name");
    fsm.AddVector3Variable("new variable name");
    fsm.AddColorVariable("new variable name");
    fsm.AddRectVariable("new variable name");
    fsm.AddQuaternionVariable("new variable name");
    fsm.AddGameObjectVariable("new variable name");

MiscCreator
^^^^^^^^^^^

Currently allows to reset everything audio of an :code:`SceneManager` and set the values of a :code:`Vector3`.

.. code-block:: c#

    var sm = GameObject.FindObjectOfType<SceneManager>();
    MiscCreator.ResetSceneManagerAudio(sm);
    sm.transform.position.Set(0, 0, 0);

USceneUtil
^^^^^^^^^^

Currently allows to search in a :code:`UnityEngine.SceneManagement.Scene` for :code:`UnityEngine.GameObject`.

.. code-block:: c#

    UnityEngine.SceneManagement.SceneManager.activeSceneChanged += OnSceneChanged;

    ...

    private void OnSceneChanged(UnityEngine.SceneManagement.Scene from, UnityEngine.SceneManagement.Scene to)
    {
        var smGo = to.FindRoot("_SceneManager");
        var someChildGo = to.Find("Whatever");
        var someOtherChildGo = smGo.Find("Whatever");
    }

Util
^^^^

Allows miscellaneous stuff and also a GetVersion function that adds hash to version number.

.. code-block:: c#

    someObject.SetAttr<ObjectClass, FieldType>("fieldName", fieldValue);
    var value = someObject.GetAttr<ObjectClass, FieldType>("fieldName");

MonoBehaviours
^^^^^^^^^^^^^^

+-------------------+----------------------------------------------------------------------------+
| Name              | Functionality                                                              |
+===================+============================================================================+
| BlurPlanePatcher  | Fixes the blurplane in custom scenes that don't include their own shaders. |
+-------------------+----------------------------------------------------------------------------+
| PatchMusicRegions | Fixes MusicRegions for easy use of custom BGM.                             |
+-------------------+----------------------------------------------------------------------------+
| SceneMapPatcher   | Fixes scenemap in custom scenes that don't include their own shaders.      |
+-------------------+----------------------------------------------------------------------------+
| SpritePatcher     | Fixes sprites in custom scenes that don't include their own shaders.       |
+-------------------+----------------------------------------------------------------------------+

Generic Mods
^^^^^^^^^^^^

In the modding api for the 1.5 version of hollow knight, generic mods will be removed, so they're added in SFCore.

+-------------------+-------------------------------------+
| Name              | Summary                             |
+===================+=====================================+
| FullSettingsMod   | A mod with save and global settings |
+-------------------+-------------------------------------+
| GlobalSettingsMod | A mod with global settings          |
+-------------------+-------------------------------------+
| SaveSettingsMod   | A mod with save settings            |
+-------------------+-------------------------------------+


.. _CharmHelper_Example: https://github.com/SFGrenade/ModdingHelper/blob/master/CharmHelper_Example.cs