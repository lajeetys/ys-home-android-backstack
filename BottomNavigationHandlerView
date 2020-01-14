import com.google.android.material.bottomnavigation.BottomNavigationView;

import java.util.ArrayList;
import java.util.LinkedHashMap;
import java.util.List;
import java.util.Map;
import java.util.Objects;
import java.util.Stack;
import java.util.concurrent.TimeUnit;

import android.annotation.SuppressLint;
import android.os.Bundle;
import android.os.Handler;
import android.widget.RelativeLayout;

import javax.inject.Inject;

import butterknife.BindView;
import butterknife.ButterKnife;

import static com.AppConstants.ACTION;
import static com.AppConstants.DATA_KEY_1;
import static com.AppConstants.DATA_KEY_2;
import static com.FragmentUtils.removeFragment;
import static com.FragmentUtils.showHideTabFragment;
import static com.StackListManager.updateStackIndex;
import static com.StackListManager.updateStackToIndexFirst;
import static com.StackListManager.updateTabStackIndex;

/**
* This is a view class, which manages the bottom navigation control 
* along with this, it shows how the backstack of the fragments is managed
*/

@SuppressLint("ViewConstructor")
public class BaseView extends RelativeLayout {

    @BindView(R.id.bnv_navbar)
    BottomNavigationView bnvNavbar;

    //Variables
    private final BaseActivity activity;
    private BaseState currentState;

    //Variables - managing backstacks
    private Map<String, Stack<String>> tagStacks;
    public String currentTab;
    private List<String> stackList;
    private List<String> menuStacks;

    //Variables - maintaining the current / top most fragment which is showing up
    private Fragment currentFragment;
    
    //Variables - bottom tab base fragments + search fragment
    private Fragment homeFragment;
    private Fragment ystvHomeFragment;
    private Fragment thirdTabFragment;
    private Fragment profileFragment;
    private Fragment searchFragment;

    @Inject
    public BaseView(BaseActivity activity, BaseState initialState) {
        super(activity);
        this.activity = activity;
        this.currentState = initialState;
        inflate(getContext(), R.layout.activity_base, this);
        ButterKnife.bind(this);
    }

    public void initViews() {
        //Listener for Bottom Navigation
        bnvNavbar.setOnNavigationItemSelectedListener(onNavigationItemSelectedListener);

        homeFragment = HomeFragment.newInstance(true);
        ystvHomeFragment = YstvFragment.newInstance(true);
        thirdTabFragment = AwardsFragment.newInstance(null, true);
        profileFragment = ProfileFragment.newInstance(null, true);
        searchFragment = SearchFragment.newInstance(true);

        tagStacks = new LinkedHashMap<>();
        tagStacks.put(HOME_SCREEN, new Stack<>());
        tagStacks.put(YSTV_SCREEN, new Stack<>());
        tagStacks.put(AWARDS_SCREEN, new Stack<>());
        tagStacks.put(PROFILE_SCREEN, new Stack<>());
        tagStacks.put(SEARCH_SCREEN, new Stack<>());

        menuStacks = new ArrayList<>();
        menuStacks.add(HOME_SCREEN);

        stackList = new ArrayList<>();
        stackList.add(HOME_SCREEN);
        stackList.add(YSTV_SCREEN);
        stackList.add(AWARDS_SCREEN);
        stackList.add(PROFILE_SCREEN);
        stackList.add(SEARCH_SCREEN);      

        bnvNavbar.setSelectedItemId(R.id.nav_home);

        //Listener for Bottom Navigation - Reselection
        bnvNavbar.setOnNavigationItemReselectedListener(onNavigationItemReselectedListener);
    }

    private final BottomNavigationView.OnNavigationItemSelectedListener onNavigationItemSelectedListener = item -> {
        switch (item.getItemId()) {
            case R.id.nav_home:
                selectedTab(HOME_SCREEN);
                return true;
            case R.id.nav_videos:
                selectedTab(YSTV_SCREEN);
                return true;
            case R.id.nav_awards:
                selectedTab(AWARDS_SCREEN);
                return true;
            case R.id.nav_profile:
                selectedTab(PROFILE_SCREEN);
                return true;
            case R.id.toolbar_search:
                selectedTab(SEARCH_SCREEN);
                return true;
        }
        return false;
    };

    private final BottomNavigationView.OnNavigationItemReselectedListener onNavigationItemReselectedListener = menuItem -> {
        switch (menuItem.getItemId()) {
            case R.id.nav_home:
            case R.id.nav_videos:
            case R.id.nav_awards:
            case R.id.nav_profile:
            case R.id.toolbar_search:
                popStackExceptFirst();
                break;
        }
    };

    /**
    * Helpful for going into the child frgament from the parent one in a particular tab stack
    */
    public void onFragmentInteractionCallback(Bundle bundle) {
        String action = bundle.getString(ACTION);
        if (action != null) {
            switch (action) {
                case HOME_SCREEN:
                    showFragment(bundle, HomeFragment.newInstance(false));
                    break;
                case YSTV_SCREEN:
                    showFragment(bundle, YstvFragment.newInstance(false));
                    break;
                case PROFILE_SCREEN:
                    showFragment(bundle, ProfileFragment.newInstance(null, false));
                    break;
                case SEARCH_SCREEN:
                    showFragment(bundle, SearchFragment.newInstance(false));
                    break;
                case AWARDS_SCREEN:
                case COMPANY_PROFILE_SCREEN:
                case INFLUENCER_PROFILE_SCREEN:
                    if (!CommonUtils.isStringNullOrEmpty(currentState.getAwardSlug())) {
                        showFragment(bundle, AwardsFragment
                                .newInstance(currentState.getAwardSlug(), false));
                    } else if (!CommonUtils.isStringNullOrEmpty(currentState.getCompanySlug())) {
                        showFragment(bundle, CompanyProfileFragment
                                .newInstance(currentState.getCompanySlug()));
                    } else if (!CommonUtils.isStringNullOrEmpty(currentState.getInfluencerSlug())) {
                        showFragment(bundle, InfluencerProfileFragment
                                .newInstance(currentState.getInfluencerSlug()));
                    } else {
                        showFragment(bundle, AwardsFragment.newInstance(null, false));
                    }
                    break;
            }
        }
    }

    /*
     * Add a fragment to the stack of a particular tab
     */
    private void showFragment(Bundle bundle, Fragment fragmentToAdd) {
        String tab = bundle.getString(DATA_KEY_1);
        boolean shouldAdd = bundle.getBoolean(DATA_KEY_2);
        FragmentUtils.addShowHideFragment(activity.getSupportFragmentManager(), tagStacks, tab, fragmentToAdd,
                getCurrentFragmentFromShownStack(), R.id.base_fragment, shouldAdd);
        assignCurrentFragment(fragmentToAdd);
    }

    private Fragment getCurrentFragmentFromShownStack() {
        return activity
                .getSupportFragmentManager()
                .findFragmentByTag(Objects.requireNonNull(tagStacks.get(currentTab))
                        .elementAt(Objects
                                .requireNonNull(tagStacks.get(currentTab)).size() - 1));
    }

    private void assignCurrentFragment(Fragment current) {
        currentFragment = current;
    }
    
    private void selectedTab(String tabId) {
        currentTab = tabId;
        RootFragment.setCurrentTab(currentTab);

        if (tagStacks == null || tagStacks.get(tabId) == null) return;

        if (Objects.requireNonNull(tagStacks.get(tabId)).size() == 0) {
            /*
              First time this tab is selected. So add first fragment of that tab.
              We are adding a new fragment which is not present in stack. So add to stack is true.
             */
            switch (tabId) {
                case HOME_SCREEN:
                    FragmentUtils.addInitialTabFragment(activity.getSupportFragmentManager(),
                            tagStacks, HOME_SCREEN, homeFragment, R.id.base_fragment,
                            true);
                    resolveStackLists(tabId);
                    assignCurrentFragment(homeFragment);
                    break;
                case YSTV_SCREEN:
                    FragmentUtils.addAdditionalTabFragment(activity.getSupportFragmentManager(),
                            tagStacks, YSTV_SCREEN, ystvHomeFragment, currentFragment,
                            R.id.base_fragment, true);
                    resolveStackLists(tabId);
                    assignCurrentFragment(ystvHomeFragment);
                    break;
                case AWARDS_SCREEN:
                    //Then follow normal workflow
                    if (!CommonUtils.isStringNullOrEmpty(currentState.getAwardSlug())) {
                        FragmentUtils.addAdditionalTabFragment(activity.getSupportFragmentManager(),
                                tagStacks, AWARDS_SCREEN, thirdTabFragment, currentFragment,
                                R.id.base_fragment, true);
                        resolveStackLists(tabId);
                        assignCurrentFragment(thirdTabFragment);
                    } else if (!CommonUtils.isStringNullOrEmpty(currentState.getCompanySlug())) {
                        FragmentUtils.addAdditionalTabFragment(activity.getSupportFragmentManager(),
                                tagStacks, COMPANY_PROFILE_SCREEN, thirdTabFragment, currentFragment,
                                R.id.base_fragment, true);
                        resolveStackLists(tabId);
                        assignCurrentFragment(thirdTabFragment);
                    } else if (!CommonUtils.isStringNullOrEmpty(currentState.getInfluencerSlug())) {
                        FragmentUtils.addAdditionalTabFragment(activity.getSupportFragmentManager(),
                                tagStacks, INFLUENCER_PROFILE_SCREEN, thirdTabFragment, currentFragment,
                                R.id.base_fragment, true);
                        resolveStackLists(tabId);
                        assignCurrentFragment(thirdTabFragment);
                    } else {
                        /**
                        * AWARDS_SCREEN is managed in such a way that, if the no data is passed to the frag
                        * It makes an api call and fetches the required data and initiates it.
                        */
                        FragmentUtils.addAdditionalTabFragment(activity.getSupportFragmentManager(),
                                tagStacks, AWARDS_SCREEN, thirdTabFragment, currentFragment,
                                R.id.base_fragment, true);
                        resolveStackLists(tabId);
                        assignCurrentFragment(thirdTabFragment);
                    }
                    break;
                case PROFILE_SCREEN:
                    FragmentUtils.addAdditionalTabFragment(activity.getSupportFragmentManager(),
                            tagStacks, PROFILE_SCREEN, profileFragment, currentFragment,
                            R.id.base_fragment, true);
                    resolveStackLists(tabId);
                    assignCurrentFragment(profileFragment);
                    break;
                case SEARCH_SCREEN:
                    FragmentUtils.addAdditionalTabFragment(activity.getSupportFragmentManager(),
                            tagStacks, SEARCH_SCREEN, searchFragment, currentFragment,
                            R.id.base_fragment, true);
                    resolveStackLists(tabId);
                    assignCurrentFragment(searchFragment);
                    break;
            }
        } else {
            /*
             * We are switching tabs, and target tab already has at least one fragment.
             * Show the target fragment
             */
            Fragment targetFragment = activity.getSupportFragmentManager()
                    .findFragmentByTag(Objects.requireNonNull(tagStacks.get(tabId)).lastElement());
            showHideTabFragment(activity.getSupportFragmentManager(), targetFragment,
                    currentFragment);
            resolveStackLists(tabId);
            assignCurrentFragment(targetFragment);
        }
    }

    private void resolveStackLists(String tabId) {
        updateStackIndex(stackList, tabId);
        updateTabStackIndex(menuStacks, tabId);
    }

    private void popStackExceptFirst() {
        if (Objects.requireNonNull(tagStacks.get(currentTab)).size() == 1) {
            return;
        }
        while (!Objects.requireNonNull(tagStacks.get(currentTab)).empty()
                && !Objects.requireNonNull(Objects.requireNonNull(activity.getSupportFragmentManager()
                .findFragmentByTag(Objects.requireNonNull(tagStacks.get(currentTab)).peek()))
                .getArguments()).getBoolean(EXTRA_IS_ROOT_FRAGMENT)) {
            activity.getSupportFragmentManager()
                    .beginTransaction()
                    .remove(Objects.requireNonNull(activity.getSupportFragmentManager()
                            .findFragmentByTag(Objects.requireNonNull(tagStacks.get(currentTab)).peek())));
            Objects.requireNonNull(tagStacks.get(currentTab)).pop();
        }
        Fragment fragment = activity.getSupportFragmentManager()
                .findFragmentByTag(Objects.requireNonNull(tagStacks
                        .get(currentTab)).elementAt(0));
        removeFragment(activity.getSupportFragmentManager(), fragment, currentFragment);
        assignCurrentFragment(fragment);
    }

    public void resolveBackPressed() {
        int stackValue = 0;
        if (Objects.requireNonNull(tagStacks.get(currentTab)).size() == 1) {
            Stack<String> value = tagStacks.get(stackList.get(1));
            if (Objects.requireNonNull(value).size() > 1) {
                stackValue = value.size();
                popAndNavigateToPreviousMenu();
            }
            if (stackValue <= 1) {
                if (menuStacks.size() > 1) {
                    navigateToPreviousMenu();
                } else {
                    activity.finish();
                }
            }
        } else {
            popFragment();
        }
    }

    private void popFragment() {
        /*
         * Select the second last fragment in current tab's stack,
         * which will be shown after the fragment transaction given below
         */
        String fragmentTag = Objects.requireNonNull(tagStacks.get(currentTab))
                .elementAt(Objects.requireNonNull(tagStacks.get(currentTab)).size() - 2);
        Fragment fragment = activity.getSupportFragmentManager().findFragmentByTag(fragmentTag);
        /*pop current fragment from stack */
        Objects.requireNonNull(tagStacks.get(currentTab)).pop();
        removeFragment(activity.getSupportFragmentManager(), fragment, currentFragment);
        assignCurrentFragment(fragment);
    }

    /*Pops the last fragment inside particular tab and goes to the second tab in the stack*/
    private void popAndNavigateToPreviousMenu() {
        String tempCurrent = stackList.get(0);
        currentTab = stackList.get(1);
        RootFragment.setCurrentTab(currentTab);
        bnvNavbar.setSelectedItemId(resolveTabPositions(currentTab));
        Fragment targetFragment = activity.getSupportFragmentManager()
                .findFragmentByTag(Objects.requireNonNull(tagStacks.get(currentTab)).lastElement());
        showHideTabFragment(activity.getSupportFragmentManager(), targetFragment, currentFragment);
        assignCurrentFragment(targetFragment);
        updateStackToIndexFirst(stackList, tempCurrent);
        menuStacks.remove(0);
    }

    private void navigateToPreviousMenu() {
        menuStacks.remove(0);
        currentTab = menuStacks.get(0);
        RootFragment.setCurrentTab(currentTab);
        bnvNavbar.setSelectedItemId(resolveTabPositions(currentTab));
        Fragment targetFragment = activity.getSupportFragmentManager()
                .findFragmentByTag(Objects.requireNonNull(tagStacks.get(currentTab)).lastElement());
        showHideTabFragment(activity.getSupportFragmentManager(), targetFragment, currentFragment);
        assignCurrentFragment(targetFragment);
    }

    private int resolveTabPositions(String currentTab) {
        int tabIndex = 0;
        switch (currentTab) {
            case HOME_SCREEN:
                tabIndex = R.id.nav_home;
                break;
            case YSTV_SCREEN:
                tabIndex = R.id.nav_videos;
                break;
            case AWARDS_SCREEN:
                tabIndex = R.id.nav_awards;
                break;
            case PROFILE_SCREEN:
                tabIndex = R.id.nav_profile;
                break;
        }
        return tabIndex;
    }
}
