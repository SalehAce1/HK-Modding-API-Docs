Utility Mod SFcore
=====================================

Background
^^^^^^^^^^
You can add custom achievements, charms and items without having a good understanding of the various aspects that run Hollow Knight and its mods. 
Therefore, this guide recommends and expects that you have already worked with the following topics:

* Understanding the structure of a basic mod.
* Adding embedded resources to a mod.

.. note::
    These steps are made using Visual Studio as an IDE, if you're using Rider, the details might not match up.
    If you are unsure about details, ask in the Discord.

Load embedded images
^^^^^^^^^^^^^^^^^^^^

Embedded image resources can be loaded with:

.. note::
    Embedded resources have the naming format :code:`{Projectname}.Resources.{Filename_With_Extension}`.

.. code-block:: c#

    private void loadResources()
    {
        Assembly _asm = Assembly.GetExecutingAssembly();
        using (Stream s = _asm.GetManifestResourceStream("CharmHelperExample.Resources.Filename.png"))
        {
            if (s != null)
            {
                byte[] buffer = new byte[s.Length];
                s.Read(buffer, 0, buffer.Length);
                s.Dispose();

                //Create texture from bytes
                var tex = new Texture2D(2, 2);

                tex.LoadImage(buffer, true);

                // Create sprite from texture
                testSprite = Sprite.Create(tex, new Rect(0, 0, tex.width, tex.height), new Vector2(0.5f, 0.5f));
            }
        }
    }

Add a reference
^^^^^^^^^^^^^^^
1) After downloading SFCore from the ModInstaller, open your project and right-click :code:`References` and click on :code:`Add Reference`

2) Select :code:`Browse...` and navigate to the :code:`Mods` folder, where the mod has been installed, after which you can :code:`Add` it.

3) Click :code:`ok`.

Add custom charms
^^^^^^^^^^^^^^^^^

1) After adding the reference, you can follow the CharmHelper_Example_ code.

2) Going through the example:

3) Preparation of the save settings of the mod to hold the data for 4 charms. (Lines 13 - 20)

.. code-block:: c#

    public class CHESaveSettings : ModSettings
    {
        // insert default values here
        public List<bool> gotCharms = new List<bool>() { true, true, true, true };
        public List<bool> newCharms = new List<bool>() { false, false, false, false };
        public List<bool> equippedCharms = new List<bool>() { false, false, false, false };
        public List<int> charmCosts = new List<int>() { 1, 1, 1, 1 };
    }

4) Add the CharmHelper as a member of the main mod class (Lines 26 - 30)

.. code-block:: c#

    public class CharmHelperExample : Mod<CHESaveSettings, CHEGlobalSettings>
    {
        //private Sprite testSprite;
        public CharmHelper charmHelper { get; private set; }
    }

5) Initialize the CharmHelper with 4 custom charms and empty sprites, also initialize the callbacks needed. (Lines 54 - 66)

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

6) Initialize the callbacks needed. (Lines 83 - 93)

.. code-block:: c#

    private void initCallbacks()
    {
        ModHooks.Instance.GetPlayerBoolHook += OnGetPlayerBoolHook;
        ModHooks.Instance.SetPlayerBoolHook += OnSetPlayerBoolHook;
        ModHooks.Instance.GetPlayerIntHook += OnGetPlayerIntHook;
        ModHooks.Instance.SetPlayerIntHook += OnSetPlayerIntHook;
        ModHooks.Instance.AfterSavegameLoadHook += initSaveSettings;
        ModHooks.Instance.ApplicationQuitHook += SaveCHEGlobalSettings;
        ModHooks.Instance.LanguageGetHook += OnLanguageGetHook;
    }

7) Form the callbacks for language. (Lines 101 - 124)

.. code-block:: c#

    private string OnLanguageGetHook(string key, string sheet)
    {
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
        return Language.Language.GetInternal(key, sheet);
    }

8) Form the callbacks for boolean checks. (Lines 126 - 197)

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

9) Form the callbacks for integer checks. (Lines 199 - 228)

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

Add custom achievements
^^^^^^^^^^^^^^^^^^^^^^^

1) After adding the reference, you can follow the CharmHelper_Example_ code, but you can leave out a lot, as most things are handled by the helper.

2) Initialize the AchievementHelper with 1 custom achievement and empty sprites.

.. note::
    This step is the one where you can add your custom sprites. Simply load those instead of the :code:`new Sprite()`.

.. note::
    For the :code:`Convo`'s to work properly, you need the :code:`ModHooks.Instance.LanguageGetHook` similar to the Helper above, but only listening to the custom convo keys.

.. code-block:: c#

    public override void Initialize()
    {
        //loadResources();

        AchievementHelper.Initialize();
        AchievementHelper.Add("YourCustomAchievementKey", new Sprite(), "YourCustomLanguageConvo", "YourCustomLanguageConvo", false);
    }

3) Done! Now you can at some point in your mod have :code:`GameManager.instance.AwardAchievement("YourCustomAchievementKey");` to grant the player the achievement.

Add custom inventory items
^^^^^^^^^^^^^^^^^^^^^^^^^^

1) After adding the reference, you can follow the CharmHelper_Example_ code, but you can leave out a lot, as most things are handled by the helper.

2) Initialize the ItemHelper with custom items and empty sprites.

.. note::
    This step is the one where you can add your custom sprites. Simply load those instead of the :code:`new Sprite()`.

.. note::
    For the :code:`Convo`'s to work properly, you need the :code:`ModHooks.Instance.LanguageGetHook` similar to the Helper above, but only listening to the custom convo keys.

.. note::
    For the :code:`playerdataBool` to work properly, you need the :code:`ModHooks.Instance.GetPlayerBoolHook` & :code:`ModHooks.Instance.SetPlayerBoolHook` similar to the CharmHelper, but only listening to the custom bool key.

.. note::
    For the :code:`playerdataInt` to work properly, you need the :code:`ModHooks.Instance.GetPlayerIntHook` & :code:`ModHooks.Instance.SetPlayerIntHook` similar to the CharmHelper, but only listening to the custom int key.

.. code-block:: c#

    public override void Initialize()
    {
        //loadResources();

        // Normal Items, like the Kings Brand, Crystal Heart, etc.
        ItemHelper.AddNormalItem("YourUniqueStateName", new Sprite(), "YourCustomPlayerDataBool", "YourCustomLanguageConvo", "YourCustomLanguageConvo");

        // Counted Items, like Simple Keys, Rancid Eggs, etc.
        ItemHelper.AddCountedItem("YourUniqueStateName", new Sprite(), "YourCustomPlayerDataInt", "YourCustomLanguageConvo", "YourCustomLanguageConvo");

        // 1 2 Both Items, like the Map, Quill and Map and Quill
        SFCore.ItemHelper.AddOneTwoBothItem("YourUniqueStateName",
            new Sprite(), new Sprite(), new Sprite(), // Sprites
            "YourCustomPlayerDataBool", "YourCustomPlayerDataBool", // PlayerData Bools
            "YourCustomLanguageConvo", "YourCustomLanguageConvo", "YourCustomLanguageConvo", // Name Convos
            "YourCustomLanguageConvo", "YourCustomLanguageConvo", "YourCustomLanguageConvo"); // Description Convos
    }

3) Done! You can now have custom Inventory Items.

Add custom enviroment particles
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1) After adding the reference, you can follow the CharmHelper_Example_ code, but you can leave out a lot, as most things are handled by the helper.

2) Initialize the EnviromentParticleHelper with custom particles. We will use enviroment type 7, which has no sprites but the same audio as grass enviroment. As such we won't be adding custom audio.

.. note::
    This step is the one where you can add your custom sprites. Simply load those instead of the :code:`new Sprite()`.

.. note::
    Currently this is rather tedious to add these effects, so this method may be changed in a future update.

.. code-block:: c#

    public override void Initialize()
    {
        //loadResources();

        EnviromentParticleHelper.Init();
        GameManager.instance.StartCoroutine(AsyncAddDashEffect(7));
        GameManager.instance.StartCoroutine(AsyncAddHardLandEffect(7));
        GameManager.instance.StartCoroutine(AsyncAddJumpEffect(7));
        GameManager.instance.StartCoroutine(AsyncAddSoftLandEffect(7));
        GameManager.instance.StartCoroutine(AsyncAddRunEffect(7));
    }
    private static IEnumerator AsyncAddDashEffect(int envType)
    {
        yield return new WaitWhile(() => !(HeroController.instance));
        yield return new WaitWhile(() => !(HeroController.instance.backDashPrefab));
        yield return new WaitWhile(() => !(HeroController.instance.backDashPrefab.GetComponent<DashEffect>()));
        yield return new WaitWhile(() => !(HeroController.instance.backDashPrefab.GetComponent<DashEffect>().dashGrass));

        var prefab = HeroController.instance.backDashPrefab.GetComponent<DashEffect>().dashGrass;
        var tmp = UObject.Instantiate(prefab, prefab.transform.parent);
        var tmpPSR = tmp.GetComponentInChildren<ParticleSystemRenderer>();
        var tmpPSR_M = tmpPSR.materials;
        tmpPSR_M[0].SetTexture("_MainTex", (new Sprite()).texture);
        EnviromentParticleHelper.addDashEffects(envType, tmp);
    }
    private static IEnumerator AsyncAddHardLandEffect(int envType)
    {
        yield return new WaitWhile(() => !(HeroController.instance));
        yield return new WaitWhile(() => !(HeroController.instance.hardLandingEffectPrefab));
        yield return new WaitWhile(() => !(HeroController.instance.hardLandingEffectPrefab.GetComponent<HardLandEffect>()));
        yield return new WaitWhile(() => !(HeroController.instance.hardLandingEffectPrefab.GetComponent<HardLandEffect>().grassObj));

        var prefab = HeroController.instance.hardLandingEffectPrefab.GetComponent<HardLandEffect>().grassObj;
        var tmp = UObject.Instantiate(prefab, prefab.transform.parent);
        var tmpPSR = tmp.GetComponentInChildren<ParticleSystemRenderer>();
        var tmpPSR_M = tmpPSR.materials;
        tmpPSR_M[0].SetTexture("_MainTex", (new Sprite()).texture);
        EnviromentParticleHelper.addHardLandEffects(envType, tmp);
    }
    private static IEnumerator AsyncAddJumpEffect(int envType)
    {
        yield return new WaitWhile(() => !(HeroController.instance));
        yield return new WaitWhile(() => !(HeroController.instance.jumpEffectPrefab));
        yield return new WaitWhile(() => !(HeroController.instance.jumpEffectPrefab.GetComponent<JumpEffects>()));
        yield return new WaitWhile(() => !(HeroController.instance.jumpEffectPrefab.GetComponent<JumpEffects>().grassEffects));

        var prefab = HeroController.instance.jumpEffectPrefab.GetComponent<JumpEffects>().grassEffects;
        var tmp = UObject.Instantiate(prefab, prefab.transform.parent);
        var tmpPSR = tmp.GetComponentInChildren<ParticleSystemRenderer>();
        var tmpPSR_M = tmpPSR.materials;
        tmpPSR_M[0].SetTexture("_MainTex", (new Sprite()).texture);
        EnviromentParticleHelper.addJumpEffects(envType, tmp);
    }
    private static IEnumerator AsyncAddSoftLandEffect(int envType)
    {
        yield return new WaitWhile(() => !(HeroController.instance));
        yield return new WaitWhile(() => !(HeroController.instance.softLandingEffectPrefab));
        yield return new WaitWhile(() => !(HeroController.instance.softLandingEffectPrefab.GetComponent<SoftLandEffect>()));
        yield return new WaitWhile(() => !(HeroController.instance.softLandingEffectPrefab.GetComponent<SoftLandEffect>().grassEffects));

        var prefab = HeroController.instance.softLandingEffectPrefab.GetComponent<SoftLandEffect>().grassEffects;
        var tmp = UObject.Instantiate(prefab, prefab.transform.parent);
        var tmpPSR = tmp.GetComponentInChildren<ParticleSystemRenderer>();
        var tmpPSR_M = tmpPSR.materials;
        tmpPSR_M[0].SetTexture("_MainTex", (new Sprite()).texture);
        EnviromentParticleHelper.addSoftLandEffects(envType, tmp);
    }
    private static IEnumerator AsyncAddRunEffect(int envType)
    {
        yield return new WaitWhile(() => !(HeroController.instance));
        yield return new WaitWhile(() => !(HeroController.instance.runEffectPrefab));

        var tmpPrefab = HeroController.instance.runEffectPrefab.transform.GetChild(1).gameObject;
        var tmp = UObject.Instantiate(tmpPrefab, tmpPrefab.transform.parent);
        var tmpPSR = tmp.GetComponent<ParticleSystemRenderer>();
        var tmpPSR_M = tmpPSR.materials;
        tmpPSR_M[0].SetTexture("_MainTex", (new Sprite()).texture);
        EnviromentParticleHelper.addRunEffects(envType, tmp);
    }

3) Done! You can now have custom enviroment particles.

Add custom menu styles
^^^^^^^^^^^^^^^^^^^^^^

1) After adding the reference, you can follow the CharmHelper_Example_ code, but you can leave out a lot, as most things are handled by the helper.

2) Initialize the MenuStyleHelper with a "custom" menu theme. We make an unused menu style avaiable and also center the gameobjects of that style.

.. note::
    This step is is being done best in the constructor of your mod class.

.. note::
    This can utilize custom logos. Look for later tutorials on how to add those.

.. code-block:: c#

    public ModName()
    {
        //loadResources();

        MenuStyleHelper.Initialize();
        MenuStyleHelper.AddMenuStyleHook += AddMyMenuStyle;
    }
    // this auto-generates, but you can leave the variable names out to save space
    private (string languageString, GameObject styleGo, int titleIndex, string unlockKey, string[] achievementKeys, MenuStyles.MenuStyle.CameraCurves cameraCurves, AudioMixerSnapshot musicSnapshot) AddMyMenuStyle(MenuStyles self)
    {
        GameObject menuStylesGo = self.gameObject;
        var radiantStyleGo = menuStylesGo.transform.GetChild(4).gameObject;
        foreach (var sr in radiantStyleGo.GetComponentsInChildren<SpriteRenderer>())
        {
            var tmpColor = sr.color;
            tmpColor.r *= 0.75f;
            tmpColor.g *= 0.75f;
            tmpColor.b *= 0.75f;
            sr.color = tmpColor;
        }
        foreach (var ps in radiantStyleGo.GetComponentsInChildren<ParticleSystem>())
        {
            var main = ps.main;
            var tmpGrad = main.startColor;
            var tmpColor = tmpGrad.colorMin;
            tmpColor.r *= 0.75f;
            tmpColor.g *= 0.75f;
            tmpColor.b *= 0.75f;
            tmpGrad.colorMin = tmpColor;
            tmpColor = tmpGrad.colorMax;
            tmpColor.r *= 0.75f;
            tmpColor.g *= 0.75f;
            tmpColor.b *= 0.75f;
            tmpGrad.colorMax = tmpColor;
            main.startColor = tmpGrad;
        }
        radiantStyleGo.transform.localPosition = new Vector3(-6.72f, 3.72f);
        radiantStyleGo.transform.GetChild(0).localPosition = new Vector3(0, -2.73f, -29.2f);
        radiantStyleGo.transform.GetChild(0).localEulerAngles = new Vector3(-90, 90, -90);
        radiantStyleGo.transform.GetChild(0).GetChild(0).localPosition = new Vector3(0, -83.9f, -0.19f);
        radiantStyleGo.transform.GetChild(0).GetChild(0).localEulerAngles = new Vector3(-90, 0, 0);
        radiantStyleGo.transform.GetChild(0).GetChild(1).localPosition = new Vector3(0, -91.6f, -0.19f);
        radiantStyleGo.transform.GetChild(0).GetChild(1).localEulerAngles = new Vector3(-90, 0, 29.95f);
        radiantStyleGo.transform.GetChild(0).GetChild(2).localPosition = new Vector3(0, -163.5f, -0.19f);
        radiantStyleGo.transform.GetChild(0).GetChild(2).localEulerAngles = new Vector3(-90, 0, 67.2f);
        radiantStyleGo.transform.GetChild(1).localPosition = new Vector3(0, 0, -1.145f);
        radiantStyleGo.transform.GetChild(1).localEulerAngles = new Vector3(0, 0, 0);
        radiantStyleGo.transform.GetChild(1).GetChild(0).localPosition = new Vector3(0, -9, 47.5f);
        radiantStyleGo.transform.GetChild(1).GetChild(0).localEulerAngles = new Vector3(0, 0, -192.483f);
        radiantStyleGo.transform.GetChild(2).localPosition = new Vector3(0, 7.4f, 21.21f);
        radiantStyleGo.transform.GetChild(3).localPosition = new Vector3(0, -32.4f, 103.2f);
        radiantStyleGo.transform.GetChild(4).localPosition = new Vector3(0, -2.7f, 145.33f);
        radiantStyleGo.transform.GetChild(5).localPosition = new Vector3(0, -4.22f, 142.91f);

        GameObject audioGo = UObject.Instantiate(self.styles[4].styleObject.transform.GetChild(8).gameObject, radiantStyleGo.transform);
        audioGo.transform.position = Vector3.zero;
        AudioSource aSource = audioGo.GetComponent<AudioSource>();
        aSource.clip = null;
        foreach (var ac in Resources.FindObjectsOfTypeAll<AudioClip>())
        {
            if (ac.name == "dream_dialogue_loop")
            {
                aSource.clip = ac;
                break;
            }
        }
        aSource.volume = 0.5f;

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

3) Done! You can now have custom menu styles.

Add custom title logos
^^^^^^^^^^^^^^^^^^^^^^

1) After adding the reference, you can follow the CharmHelper_Example_ code, but you can leave out a lot, as most things are handled by the helper.

2) Initialize the TitleLogoHelper with a logo spite.

.. note::
    This step is is being done best in the constructor of your mod class.

.. note::
    This step is the one where you can add your custom sprites. Simply load those instead of the :code:`new Sprite()`.

.. code-block:: c#

    private int LogoId = -1;
    public ModName()
    {
        //loadResources();

            TitleLogoHelper.Initialize();
            LogoId = TitleLogoHelper.AddLogo(new Sprite());
    }

3) Done! You can now have custom title logos.


.. _CharmHelper_Example: https://github.com/SFGrenade/ModdingHelper/blob/master/CharmHelper_Example.cs
.. _Add_custom_charms: https://radiance.host/apidocs/SFCore.html#add-custom-charms