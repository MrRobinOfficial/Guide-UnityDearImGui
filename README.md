# Guide-UnityDearImGui

Code for using <em>Dear ImGui</em> solution with <em>Unity</em>

# Repos

[realgamessoftware - Original Repo](https://github.com/realgamessoftware/dear-imgui-unity)<br>
[mattmanj17 - Forked Repo](https://github.com/mattmanj17/dear-imgui-unity)

Forked repo fixes an issue from Unity version 2020.1.0b4: https://github.com/realgamessoftware/dear-imgui-unity/issues/28

# How To Install

* [Add package](https://docs.unity3d.com/Manual/upm-ui-giturl.html) from this git URL: https://github.com/mattmanj17/dear-imgui-unity.git
* Add a DearImGui component to one of the objects in the scene.
* Create 'Ini Settings Asset' and 'Font Atlas Config Asset' in the project folder.
* Add custom fonts to the 'StreamingAssets' folder and link it up to the 'Font Atlas Config Asset'.
* Change the 'Font pixel' in the 'Font Atlas Config Asset', to scale up the font size.
* When using the Universal Render Pipeline, add a Render Im Gui Feature render feature to the renderer asset. Assign it to the render feature field of the DearImGui component.
* Subscribe to the ImGuiUn.Layout event and use ImGui functions.
* Example script:

```.c#
using UnityEngine;
using ImGuiNET;

public class DearImGuiDemo : MonoBehaviour
{
    void OnEnable()
    {
        ImGuiUn.Layout += OnLayout;
    }

    void OnDisable()
    {
        ImGuiUn.Layout -= OnLayout;
    }

    void OnLayout()
    {
        ImGui.ShowDemoWindow();
    }
}
```

# Advanced Script

```.c#
using ImGuiNET;
using UnityEngine;
using UnityEngine.Events;

public class DearImGuiDemoWindow : MonoBehaviour
{
    private static UnityAction OnUpdateLighting;
    private static UnityAction OnLoadGame;
    private static UnityAction OnSaveGame;

    private void OnEnable() => ImGuiUn.Layout += OnLayout;

    private void OnDisable() => ImGuiUn.Layout -= OnLayout;

    private static readonly Vector4 TEXT_COLOR = new(1f, 0.8f, 0f, 1.0f);

    private float m_Timeline = 0.5f;
    private System.TimeSpan m_Time = System.TimeSpan.FromHours(12.0f);
    private string m_Username = string.Empty;
    private string m_Password = string.Empty;
    private Vector3 m_SunColor = new(1.0f, 0.9f, 0f);
    private bool m_WindowEnabled = false;
    private bool m_EnableDayAndNightCycle = true;
    private int m_DragInt = 0;
    private int m_AccountCount = 0;
    private bool m_ShowImGuiDemoWindow;
    private static uint s_tab_bar_flags = (uint)ImGuiTabBarFlags.Reorderable;
    static bool[] s_opened = { true, true, true, true }; // Persistent user state

    /**
     * DearImGui
     * Manual - https://pthom.github.io/imgui_manual_online/manual/imgui_manual.html
     */


    private void OnLayout()
    {
        // Begins ImGui window
        if (!ImGui.Begin("Game Manager",
                         ref m_WindowEnabled,
                         ImGuiWindowFlags.MenuBar))
            return;

        ImGui.Checkbox("Enable Day And Night Cycle", ref m_EnableDayAndNightCycle);

        // Make a float slider (label, value, min, max) 
        if (ImGui.SliderFloat("Time [%]", ref m_Timeline, 0f, 1f) && m_EnableDayAndNightCycle)
        {
            m_Time = System.TimeSpan.FromSeconds(m_Timeline * 86400f);
            OnUpdateLighting?.Invoke();
        }

        // Display text of current in-game time
        ImGui.TextColored(TEXT_COLOR, $"In-game Time: {m_Time:hh\\:mm}");

        // Create color editor for Vector3
        ImGui.ColorEdit3("Sun Color", ref m_SunColor);

        // Creates a menu bar
        if (ImGui.BeginMenuBar())
        {
            if (ImGui.BeginMenu("File"))
            {
                if (ImGui.MenuItem("Open", shortcut: "Ctrl+O"))
                    Debug.Log("Opening a file...");

                if (ImGui.MenuItem("Save", shortcut: "Ctrl+S"))
                    Debug.Log("Saving a file...");

                if (ImGui.MenuItem("Close", shortcut: "Ctrl+W"))
                {
                    m_WindowEnabled = false;
                    Debug.Log("Closing this window...");
                }

                ImGui.EndMenu();
            }

            ImGui.EndMenuBar();
        }

        if (ImGui.Button("Load Game"))
        {
            OnLoadGame?.Invoke();
            Debug.Log("Loading the game...");
        }

        if (ImGui.Button("Save Game"))
        {
            OnSaveGame?.Invoke();
            Debug.Log("Saving the game...");
        }

        if (ImGui.Button("Create Account"))
        {
            Debug.Log("Creating account...");
            m_AccountCount++;
        }

        ImGui.SameLine(0, -1);
        ImGui.Text($"Account Count = {m_AccountCount}");

        // Create input field (label, value, maxLength [uint])
        ImGui.InputText("Username", ref m_Username, maxLength: 12u);
        ImGui.InputText("Password", ref m_Password, maxLength: 16u);

        ImGui.Text($"Mouse position: {ImGui.GetMousePos()}");

        // Display contents in a scrolling region
        ImGui.TextColored(TEXT_COLOR, "Important Stuff");
        ImGui.BeginChild("Scrolling");

        for (int n = 0; n < 50; n++)
            ImGui.Text($"{n:0000}: Some text");

        ImGui.EndChild();

        // Generate samples and plot them
        float[] samples = new float[100];

        for (int n = 0; n < 100; n++)
            samples[n] = Mathf.Sin((float)(n * 0.2f + ImGui.GetTime() * 1.5f));

        ImGui.PlotLines("Samples", ref samples[0], 100);

        ImGui.DragInt("Draggable Int", ref m_DragInt);

        float framerate = ImGui.GetIO().Framerate;
        ImGui.Text($"Application average {1000.0f / framerate:0.##} ms/frame ({framerate:0.#} FPS)");

        if (m_ShowImGuiDemoWindow)
        {
            // Normally user code doesn't need/want to call this because positions are saved in .ini file anyway.
            // Here we just want to make the demo initial state a bit more friendly!
            ImGui.SetNextWindowPos(new Vector2(650, 20), ImGuiCond.FirstUseEver);
            ImGui.ShowDemoWindow(ref m_ShowImGuiDemoWindow);
        }

        if (ImGui.TreeNode("Tabs"))
        {
            if (ImGui.TreeNode("Basic"))
            {
                ImGuiTabBarFlags tab_bar_flags = ImGuiTabBarFlags.None;

                if (ImGui.BeginTabBar("MyTabBar", tab_bar_flags))
                {
                    if (ImGui.BeginTabItem("Avocado"))
                    {
                        ImGui.Text("This is the Avocado tab!\nblah blah blah blah blah");
                        ImGui.EndTabItem();
                    }

                    if (ImGui.BeginTabItem("Broccoli"))
                    {
                        ImGui.Text("This is the Broccoli tab!\nblah blah blah blah blah");
                        ImGui.EndTabItem();
                    }

                    if (ImGui.BeginTabItem("Cucumber"))
                    {
                        ImGui.Text("This is the Cucumber tab!\nblah blah blah blah blah");
                        ImGui.EndTabItem();
                    }

                    ImGui.EndTabBar();
                }

                ImGui.Separator();
                ImGui.TreePop();
            }

            if (ImGui.TreeNode("Advanced & Close Button"))
            {
                // Expose a couple of the available flags. In most cases you may just call BeginTabBar() with no flags (0).
                ImGui.CheckboxFlags("ImGuiTabBarFlags_Reorderable", ref s_tab_bar_flags, (uint)ImGuiTabBarFlags.Reorderable);
                ImGui.CheckboxFlags("ImGuiTabBarFlags_AutoSelectNewTabs", ref s_tab_bar_flags, (uint)ImGuiTabBarFlags.AutoSelectNewTabs);
                ImGui.CheckboxFlags("ImGuiTabBarFlags_NoCloseWithMiddleMouseButton", ref s_tab_bar_flags, (uint)ImGuiTabBarFlags.NoCloseWithMiddleMouseButton);
                if ((s_tab_bar_flags & (uint)ImGuiTabBarFlags.FittingPolicyMask) == 0)
                    s_tab_bar_flags |= (uint)ImGuiTabBarFlags.FittingPolicyDefault;
                if (ImGui.CheckboxFlags("ImGuiTabBarFlags_FittingPolicyResizeDown", ref s_tab_bar_flags, (uint)ImGuiTabBarFlags.FittingPolicyResizeDown))
                    s_tab_bar_flags &= ~((uint)ImGuiTabBarFlags.FittingPolicyMask ^ (uint)ImGuiTabBarFlags.FittingPolicyResizeDown);
                if (ImGui.CheckboxFlags("ImGuiTabBarFlags_FittingPolicyScroll", ref s_tab_bar_flags, (uint)ImGuiTabBarFlags.FittingPolicyScroll))
                    s_tab_bar_flags &= ~((uint)ImGuiTabBarFlags.FittingPolicyMask ^ (uint)ImGuiTabBarFlags.FittingPolicyScroll);

                // Tab Bar
                string[] names = { "Artichoke", "Beetroot", "Celery", "Daikon" };

                for (int n = 0; n < s_opened.Length; n++)
                {
                    if (n > 0) { ImGui.SameLine(); }
                    ImGui.Checkbox(names[n], ref s_opened[n]);
                }

                // Passing a bool* to BeginTabItem() is similar to passing one to Begin(): the underlying bool will be set to false when the tab is closed.
                if (ImGui.BeginTabBar("MyTabBar", (ImGuiTabBarFlags)s_tab_bar_flags))
                {
                    for (int n = 0; n < s_opened.Length; n++)
                    {
                        if (s_opened[n] && ImGui.BeginTabItem(names[n], ref s_opened[n]))
                        {
                            ImGui.Text($"This is the {names[n]} tab!");
                            if ((n & 1) != 0)
                                ImGui.Text("I am an odd tab.");
                            ImGui.EndTabItem();
                        }
                    }

                    ImGui.EndTabBar();
                }

                ImGui.Separator();
                ImGui.TreePop();
            }

            ImGui.TreePop();
        }

        // Ends ImGui window
        ImGui.End();
    }
}
```

# Helpful links

[Dear ImGui Manual In C++](https://pthom.github.io/imgui_manual_online/manual/imgui_manual.html)<br/>
[realgamessoftware - Original Repo](https://github.com/realgamessoftware/dear-imgui-unity/issues)<br>
[mattmanj17 - Forked Repo](https://github.com/mattmanj17/dear-imgui-unity)<br>
[ImGui.NET - Repo](https://github.com/mellinoe/ImGui.NET)<br>
[cimgui - Repo](https://github.com/cimgui/cimgui)<br>
[imgui - 'Creator' Repo](https://github.com/ocornut/imgui)<br>

# See Also
This package uses Dear ImGui C bindings by [cimgui](https://github.com/cimgui/cimgui) and the C# wrapper by [ImGui.NET](https://github.com/mellinoe/ImGui.NET).

The development project for the package can be found at https://github.com/realgamessoftware/dear-imgui-unity-dev .
